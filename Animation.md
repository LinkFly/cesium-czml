Cesium / Анимация
---------------

**Анимация через `SampledProperty`**

Для того, чтобы создать анимацию, можно воспользоваться классом Cesium.SampledProperty.
С помощью этого класса создаётся объект, представляющий собой изменяющееся во 
времени значение. Для создания и настройки экземпляра `SampledProperty` необходимы
следующие шаги:
 - Создать экземпляр класса `SampledProperty` с указанием типа значения, 
 которое будет изменяться. Например:
 
   `let pulseProperty = new Cesium.SampledProperty(Number);`
   
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
pulseProperty.addSample(timePoint1, 1);
pulseProperty.addSample(timePoint2, 5);
```
 
 - Теперь с помощью "изменяющего во времени значения" (экземпляра класса 
 `SampledProperty`) мы можем получить значение находящееся между опорными
  временными точками:
 
```javascript
pulseProperty.getValue(timePointMid)
```
 - Полный пример выглядит так:
 
```javascript
let pulseProperty = new Cesium.SampledProperty(Number);
let timePoint1 = Cesium.JulianDate.fromIso8601('2021-02-10T00:00:00.00Z');
let timePoint2 = Cesium.JulianDate.fromIso8601('2021-02-10T00:01:00.00Z');
pulseProperty.addSample(timePoint1, 1);
pulseProperty.addSample(timePoint2, 5);
console.log('Interpolated value: ',
    pulseProperty.getValue(Cesium.JulianDate.fromIso8601('2021-02-10T00:00:30.00Z')));
// >> Interpolated value:  3
``` 

 - А также, можно получить интерполяцию значений координат, например:
 ```javascript
let timePoint1 = Cesium.JulianDate.fromIso8601('2021-02-10T00:00:00.00Z');
let timePoint2 = Cesium.JulianDate.fromIso8601('2021-02-10T00:01:00.00Z');
let coordsSamplePropery = new Cesium.SampledProperty(Cesium.Cartesian3);
let vals_coords_points = [
    new Cesium.Cartesian3(
            0,
            10,
            20
    ),
    new Cesium.Cartesian3(
            10,
            20,
            40
    )
];
coordsSamplePropery.addSample(timePoint1, vals_coords_points[0]);
coordsSamplePropery.addSample(timePoint2, vals_coords_points[1]);
console.log('Interpolated value: ',
    coordsSamplePropery.getValue(Cesium.JulianDate.fromIso8601('2021-02-10T00:00:30.00Z')));
// >> Interpolated value:  Cartesian3{x: 5, y: 15, z: 30}
```
 - После создания изменяющего во времени значения (после получения экземпляра Cesium.SampledProperty),
   алгоритм интерполяции можно настроить (также имеются следующи алгоритмы интерполяции:
   LagrangePolynomialApproximation, LinearApproximation - по-умолчанию):
```javascript
var pulseCoordsProperty = new Cesium.SampledProperty(Cesium.Cartesian3);
pulseCoordsProperty.setInterpolationOptions({
  interpolationDegree : 3,
  interpolationAlgorithm : Cesium.HermitePolynomialApproximation
});


```

    
 
