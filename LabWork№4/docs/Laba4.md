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
- Скрин из постмана:
  ![image](https://github.com/Olezha228/PAPS.LABA_1/assets/87082100/5a0159a2-fadf-44dd-b624-bee01b36916d)

- Скрин рез-тов тестов:
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
  …    
      responseData.forEach(function(well) {
          pm.expect(well).to.have.property(sortFieldParam);
      });
  });
  ```
- результаты тестов:
  ![image](https://github.com/Olezha228/PAPS.LABA_1/assets/87082100/5d51eeb8-643e-4ff6-9876-fbb5200a370e)



