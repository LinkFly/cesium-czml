Cesium / Анимация
---------------

**Анимация через `SampledProperty`**

Для того, чтобы создать анимацию, можно воспользоваться классом Cesium.SampledProperty.
С помощью этого класса создаётся объект, представляющий собой изменяющееся во 
времени значение. Для создания и настройки экземпляра `SampledProperty` необходимы
следующие шаги:
 - Создать экземпляр класса `SampledProperty` с указанием типа значения, 
 которое будет изменяться. Например:
 
   `let sProp = new Cesium.SampledProperty(Number);`
   
 - Создать интересующие опорные точки во времени (далее с этими точками 
 будут связываться значения). Тип времени должен быть представлен классом 
 `Cesium.JulianDate`. Экземляр этого класса можно получить через конвертацию
 строки с датой в формате "ISO-8601" с помощью метода `JulianDate.fromIso8601`.
 Например:
 
 ```javascript
var timePoint1 = Cesium.JulianDate.fromIso8601('2021-02-10T00:00:00.00Z');
var timePoint2 = Cesium.JulianDate.fromIso8601('2021-02-10T00:01:00.00Z');
```
 
 - Далее, необходимо связать точки времени со значениями (это подобно установке
 ключевых кадров):
 
 ```javascript
sProp.addSample(timePoint1, 1);
sProp.addSample(timePoint2, 5);
```
 
 - Теперь с помощью "изменяющего во времени значения" (экземпляра класса 
 `SampledProperty`) мы можем получить значение находящееся между опорными
  временными точками:
 
```javascript
sProp.getValue(timePointMid)
```
 - Полный пример выглядит так:
 
```javascript
var sProp = new Cesium.SampledProperty(Number);
var timePoint1 = Cesium.JulianDate.fromIso8601('2021-02-10T00:00:00.00Z');
var timePoint2 = Cesium.JulianDate.fromIso8601('2021-02-10T00:01:00.00Z');
sProp.addSample(timePoint1, 1);
sProp.addSample(timePoint2, 5);
console.log('Interpolated value: ',
    sProp.getValue(Cesium.JulianDate.fromIso8601('2021-02-10T00:00:30.00Z')));
// >> Interpolated value:  3
``` 

