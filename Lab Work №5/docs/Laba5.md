Так как мое приложение не основано на микросервисной архитектуре, то я сделал просто пример микросервисов.

## 1 микросервис
Данный микросервис имеет метод Get, который возвращает определенный результат.
```c#
        [HttpGet]
        public IActionResult Get()
        {
            var data = new { Name = "John", Age = 30 };
            return Ok(data);
        }
```

Пример работы метода в swagger.
![image](https://github.com/Olezha228/PAPS.LABA_1/assets/87082100/836d74f4-03cf-4480-8c36-2d33a56e3b8b)

Докерфайл для первого мс.
```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:5.0 AS base
WORKDIR /app
EXPOSE 80

FROM mcr.microsoft.com/dotnet/sdk:5.0 AS build
WORKDIR /src
COPY ["First/First.csproj", "First/"]
RUN dotnet restore "First/First.csproj"
COPY . .
WORKDIR "/src/First"
RUN dotnet build "First.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "First.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "First.dll"]
```



## 2 микросервис
2 Микросервис так же имеет метод Get, который возвращает определенный результат, за которым он обращается к 1-му микросервису.
```c#
        [HttpGet]
        public async Task<IActionResult> Get()
        {
            var response = await _httpClient.GetAsync("http://host.docker.internal:32780/Data");
            if (response.IsSuccessStatusCode)
            {
                var data = await response.Content.ReadAsStringAsync();
                return Ok(data);
            }

            return StatusCode((int)response.StatusCode, "Error fetching data from ProjectA");
        }
```

Пример работы метода в swagger.
![image](https://github.com/Olezha228/PAPS.LABA_1/assets/87082100/aaecc2c1-dd8f-417c-95e0-1f7788af9188)


Докерфайл для второго мс.
```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:5.0 AS base
WORKDIR /app
EXPOSE 80

FROM mcr.microsoft.com/dotnet/sdk:5.0 AS build
WORKDIR /src
COPY ["Second/Second.csproj", "Second/"]
RUN dotnet restore "Second/Second.csproj"
COPY . .
WORKDIR "/src/Second"
RUN dotnet build "Second.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "Second.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "Second.dll"]
```



Пример работающих контейнеров из Docker Desktop.
![image](https://github.com/Olezha228/PAPS.LABA_1/assets/87082100/896f406a-d035-49ef-8559-c60b09662e51)

Также был настроен pipeline CI/CD.
```yml
name: .NET Core Desktop

on:
  push:
    branches: [ "master" ]
  pull_request:
    branches: [ "master" ]

jobs:

  build:

    strategy:
      matrix:
        configuration: [Debug, Release]

    runs-on: windows-latest  # For a list of available runner types, refer to
                             # https://help.github.com/en/actions/reference/workflow-syntax-for-github-actions#jobsjob_idruns-on

    env:
      Solution_Name: your-solution-name                         # Replace with your solution name, i.e. MyWpfApp.sln.
      Test_Project_Path: your-test-project-path                 # Replace with the path to your test project, i.e. MyWpfApp.Tests\MyWpfApp.Tests.csproj.
      Wap_Project_Directory: your-wap-project-directory-name    # Replace with the Wap project directory relative to the solution, i.e. MyWpfApp.Package.
      Wap_Project_Path: your-wap-project-path                   # Replace with the path to your Wap project, i.e. MyWpf.App.Package\MyWpfApp.Package.wapproj.

    steps:
    - name: Checkout
      uses: actions/checkout@v3
      with:
        fetch-depth: 0

    # Install the .NET Core workload
    - name: Install .NET Core
      uses: actions/setup-dotnet@v3
      with:
        dotnet-version: 6.0.x

    # Add  MSBuild to the PATH: https://github.com/microsoft/setup-msbuild
    - name: Setup MSBuild.exe
      uses: microsoft/setup-msbuild@v1.0.2

    # Execute all unit tests in the solution
    - name: Execute unit tests
      run: dotnet test

    # Restore the application to populate the obj folder with RuntimeIdentifiers
    - name: Restore the application
      run: msbuild $env:Solution_Name /t:Restore /p:Configuration=$env:Configuration
      env:
        Configuration: ${{ matrix.configuration }}

    # Decode the base 64 encoded pfx and save the Signing_Certificate
    - name: Decode the pfx
      run: |
        $pfx_cert_byte = [System.Convert]::FromBase64String("${{ secrets.Base64_Encoded_Pfx }}")
        $certificatePath = Join-Path -Path $env:Wap_Project_Directory -ChildPath GitHubActionsWorkflow.pfx
        [IO.File]::WriteAllBytes("$certificatePath", $pfx_cert_byte)

    # Create the app package by building and packaging the Windows Application Packaging project
    - name: Create the app package
      run: msbuild $env:Wap_Project_Path /p:Configuration=$env:Configuration /p:UapAppxPackageBuildMode=$env:Appx_Package_Build_Mode /p:AppxBundle=$env:Appx_Bundle /p:PackageCertificateKeyFile=GitHubActionsWorkflow.pfx /p:PackageCertificatePassword=${{ secrets.Pfx_Key }}
      env:
        Appx_Bundle: Always
        Appx_Bundle_Platforms: x86|x64
        Appx_Package_Build_Mode: StoreUpload
        Configuration: ${{ matrix.configuration }}

    # Remove the pfx
    - name: Remove the pfx
      run: Remove-Item -path $env:Wap_Project_Directory\GitHubActionsWorkflow.pfx

    # Upload the MSIX package: https://github.com/marketplace/actions/upload-a-build-artifact
    - name: Upload build artifacts
      uses: actions/upload-artifact@v3
      with:
        name: MSIX Package
        path: ${{ env.Wap_Project_Directory }}\AppPackages
```

## Интеграционные тесты
```c#
public class IntegrationTests : IClassFixture<TestFixture<Startup>>
    {
        private readonly HttpClient _client;

        public IntegrationTests(TestFixture<Startup> fixture)
        {
            _client = fixture.Client;
        }

        [Fact]
        public async Task GetRemoteData_ReturnsDataFromProjectA()
        {
            // Arrange
            var request = "/RemoteData";

            // Act
            var response = await _client.GetAsync(request);

            // Assert
            response.EnsureSuccessStatusCode();
            var responseString = await response.Content.ReadAsStringAsync();
            Assert.Contains("John", responseString); // Assuming ProjectA/Data returns JSON with "Name" field containing "John"
            Assert.Contains("Age", responseString); // Assuming ProjectA/Data returns JSON with "Age" field
        }
    }
```
