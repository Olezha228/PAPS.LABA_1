# Лабораторная 4

## Метод GET /api/well/statistics
- Этот метод возвращает статистику по скважинам.
- Параметры запроса: Отсутствуют
- Пример запроса: GET /api/well/statistics
- Пример ответа (в json):
```json
{
  "totalWells": 50,
  "averageDepth": 1500.5
}
```

- В ответе дается информация о кол-ве скважин и средней их глубине.

- Скрин из постмана:
  ![image](https://github.com/Olezha228/PAPS.LABA_1/assets/87082100/5a0159a2-fadf-44dd-b624-bee01b36916d)

- Код теста:
  ```javascript
  pm.test("Status code is 200", function () {
      pm.response.to.have.status(200);
  });
  
  pm.test("Response has correct format and values", function () {
      pm.response.to.be.json;
      var jsonData = pm.response.json();
      pm.expect(jsonData).to.have.property('totalWells');
      pm.expect(jsonData.totalWells).to.be.a('number');
  
      pm.expect(jsonData).to.have.property('averageDepth');
      pm.expect(jsonData.averageDepth).to.be.a('number');
  });
  ```

- рез-т тестов:
  ![image](https://github.com/Olezha228/PAPS.LABA_1/assets/87082100/dc1229bd-c15f-4be6-b276-a76823f18cb8)

## Метод GET /api/well
- Этот метод возвращает список скважин.

- Параметры запроса:
  1. sortField (опционально) - поле для сортировки, может принимать значения: "name", "id", и другие.
  
- Пример запроса: GET /api/well?sortField=name
- Пример ответа:
  ```json
  [
    {
      "id": 1,
      "name": "Well 1",
      "depth": 1500
    },
    {
      "id": 2,
      "name": "Well 2",
      "depth": 1600
    },
    // ...
  ]
  ```
- скрин из постмана:
  ![image](https://github.com/Olezha228/PAPS.LABA_1/assets/87082100/17513bf7-f74d-4bb6-a4a1-9100ea5dd3de)
  ![image](https://github.com/Olezha228/PAPS.LABA_1/assets/87082100/9548d36a-1a21-44e9-9ee4-85f3baf65929)

- тесты:
  ```javascript
  pm.test("Response status code is 200", function () {
    pm.response.to.have.status(200);
  });
  
  pm.test("Response has the required fields - id, name, and depth", function () {
      const responseData = pm.response.json();
      
      pm.expect(responseData).to.be.an('array');
      responseData.forEach(function(item) {
          pm.expect(item).to.have.property('id');
          pm.expect(item).to.have.property('name');
          pm.expect(item).to.have.property('depth');
      });
  });
  
  pm.test("Name is a non-empty string", function () {
      const responseData = pm.response.json();
      
      responseData.forEach(function(well) {
          pm.expect(well.name).to.be.a('string').and.to.have.lengthOf.at.least(1, "Name should not be empty");
      });
  });
  
  pm.test("Depth is a non-negative integer", function () {
      const responseData = pm.response.json();
  
      pm.expect(responseData).to.be.an('array');
      responseData.forEach(function(well) {
          pm.expect(well.depth).to.be.a('number').and.to.be.at.least(0);
      });
  });
  
  pm.test("Sort field parameter is correctly reflected in the response", function () {
      const sortFieldParam = pm.request.url.query.get('sortField');
      const responseData = pm.response.json();
      
      responseData.forEach(function(well) {
          pm.expect(well).to.have.property(sortFieldParam);
      });
  });
  ```
- результаты тестов:
  ![image](https://github.com/Olezha228/PAPS.LABA_1/assets/87082100/5d51eeb8-643e-4ff6-9876-fbb5200a370e)

## Метод `GET /api/well/filter`

- Этот метод возвращает отфильтрованный список скважин по глубине.

- Параметры запроса:
  1. `minDepth` - минимальная глубина
  2. `maxDepth` - максимальная глубина
  
- Пример запроса: GET /api/well/filter?minDepth=400&maxDepth=700
- Пример ответа:
  ```json
  [
    {
        "id": 1,
        "name": "Well_1",
        "depth": 687
    },
    {
        "id": 10,
        "name": "Well_10",
        "depth": 700
    },
    {
        "id": 15,
        "name": "Well_15",
        "depth": 657
    },
    // ...
  ]
  ```

- скрин из постмана:
  ![image](https://github.com/Olezha228/PAPS.LABA_1/assets/87082100/04e39bb6-87b3-4062-8ea9-2e6a27858297)

- тесты:
  ```javascript
  pm.test("Response content type is application/json", function () {
    pm.expect(pm.response.headers.get("Content-Type")).to.include("application/json");
  });
  
  
  pm.test("Validate the response array structure", function () {
      const responseData = pm.response.json();
      
      pm.expect(responseData).to.be.an('array').that.is.not.empty;
  …
  pm.test("MinDepth and maxDepth parameters are correctly reflected in the response", function () {
      const responseData = pm.response.json();
  
      responseData.forEach(function(well) {
          pm.expect(well.depth).to.be.at.least(400, "Depth should be greater than or equal to minDepth");
          pm.expect(well.depth).to.be.at.most(700, "Depth should be less than or equal to maxDepth");
      });
  });
  ```

- рез-ты тестов:
  ![image](https://github.com/Olezha228/PAPS.LABA_1/assets/87082100/8b27cb92-a7a9-4d2d-9f54-6ea281bc37bc)


## Метод `POST /api/well`
- Этот метод создает новую скважину.
- 
- Параметры запроса:
  1. `name` - название скважины (в теле запроса)

- Пример запроса:
  POST /api/well
  Content-Type: application/json
  {
    "name": "New Well",
    "depth": 1800
  }

- Пример ответа:
  ```json
  {
    "id": 51,
    "name": "New Well",
    "depth": 1800
  }
  ```
- скрин из постмана:
  ![image](https://github.com/Olezha228/PAPS.LABA_1/assets/87082100/ac5ff30a-8e28-462e-8e71-82445d00d1ba)
  ![image](https://github.com/Olezha228/PAPS.LABA_1/assets/87082100/65fa9bd5-e0a7-41c1-b10a-b32ef701fb90)


- тесты:
  ```javascript
  pm.test("Response has the required fields - id, name, and depth", function () {
    const responseData = pm.response.json();
    
    pm.expect(responseData).to.be.an('object');
    pm.expect(responseData).to.have.property('id');
    pm.expect(responseData).to.have.property('name');
    pm.expect(responseData).to.have.property('depth');
  });

  pm.test("Name is a non-empty string", function () {
    const responseData = pm.response.json();
    
    pm.expect(responseData).to.be.an('object');
    pm.expect(responseData.name).to.be.a('string').and.to.have.lengthOf.at.least(1, "Name should not be empty");
  });
  
  pm.test("Depth is a non-negative integer", function () {
    const responseData = pm.response.json();
    
    pm.expect(responseData).to.be.an('object');
    pm.expect(responseData.depth).to.be.a('number');
    pm.expect(responseData.depth).to.be.at.least(0);
  });
  
  pm.test("Content-Type header is application/json", function () {
      pm.expect(pm.response.headers.get("Content-Type")).to.include("application/json");
  });
  ```

- рез-тат тестов
  ![image](https://github.com/Olezha228/PAPS.LABA_1/assets/87082100/36678437-cf04-4c7e-a8fb-c956b71eda1e)


## Метод GET /api/well/{id}

- Этот метод возвращает информацию о конкретной скважине.
- Параметры запроса:
  1. id - идентификатор скважины
 
- Пример запроса: GET /api/well/1

- Пример ответа:
  ```json
  {
    "id": 1,
    "name": "Well 1",
    "depth": 1500
  }
  ```

- Скрин из постмана:
  ![image](https://github.com/Olezha228/PAPS.LABA_1/assets/87082100/c119d0ad-6ae5-449d-9355-172d10c0082c)

- Код тестов:
  ```js
  pm.test("Response status code is 200", function () {
    pm.expect(pm.response.code).to.equal(200);
  });
  
  pm.test("Response has the required fields - id, name, and depth", function () {
      const responseData = pm.response.json();
      
      pm.expect(responseData).to.be.an('object');
      pm.expect(responseData).to.have.property('id');
      pm.expect(responseData).to.have.property('name');
      pm.expect(responseData).to.have.property('depth');
  });
  
  pm.test("Name is a non-empty string", function () {
      const responseData = pm.response.json();
      
      pm.expect(responseData).to.be.an('object');
      pm.expect(responseData.name).to.be.a('string').and.to.have.lengthOf.at.least(1, "Name should not be empty");
  });
  ```

- рез-тат тестов:
  ![image](https://github.com/Olezha228/PAPS.LABA_1/assets/87082100/48cfd71d-11a3-41ec-a822-65e16629b86b)


















