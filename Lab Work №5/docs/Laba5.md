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
