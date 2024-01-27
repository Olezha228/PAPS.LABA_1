# Лабораторная 4

## Метод GET /api/well/statistics
Этот метод возвращает статистику по скважинам.
Параметры запроса: Отсутствуют
Пример запроса: GET /api/well/statistics
Пример ответа (в json):
```json
{
  "totalWells": 50,
  "averageDepth": 1500.5
}
```
В ответе дается информация о кол-ве скважин и среднй их глубине.
Код теста:
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
Скрин из постмана:
![image](https://github.com/Olezha228/PAPS.LABA_1/assets/87082100/5a0159a2-fadf-44dd-b624-bee01b36916d)
Скрин тестов:
![image](https://github.com/Olezha228/PAPS.LABA_1/assets/87082100/dc1229bd-c15f-4be6-b276-a76823f18cb8)

