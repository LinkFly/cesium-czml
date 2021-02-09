Cesium / Анимация
====================

Анимация значений через `SampledProperty`
--------------------------------

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

 - Изменяющиеся значения можно применить к примитивам Cesium. Например при определения точки, мы
 можем задать в некоторых св-вах изменяющиеся значения (для позиции и размера):
```javascript
   viewer.entities.add({
     position: pulseCoordsProperty,
     point : {
       pixelSize: pulseProperty,
       color : Cesium.Color.RED,
       outlineWidth: 0
     }
   });
```
 
Активация анимации в сцене
----------------------------

**Получение `часов`**

Для того, чтобы активировать анимацию в сцене необходимо настроить `часы` - это экземпляр класса
`Cesium.Clock`. Его можно получить из экземляра базового виджета Cesium (`Cesium.Viewer`), сразу
после его создания:

```javascript
const viewer = new Cesium.Viewer('cesiumContainer', {
    terrainProvider: Cesium.createWorldTerrain()
  });
const clock = viewer.clock;
```

**Настройка `часов`**

 - Главное, что необходимо чтобы активировать анимацию - её включить в полученных часах:
```javascript
const clock = viewer.clock;
clock.shouldAnimate = true;
```
   Возможно имеет смысл включать анимацию, после наступления некоторого события (например
   после полной загрузки тайлов)

--------------
( in-progress: TODO - настройка полей startTime, currentTime, stopTime требует изучения и 
экспериментов, так как и без этого анимация прекрасно отработала

Для настройки часов необходимо инициализовать его поля, связанные с временем старта (startTime),
текущем времени (currentTime), времени остановки (stopTime). После того, как временной цикл закончен,
таймер вернётся в зачение startTime (соответственно первый временной цикл не обязан начинаться со
startTime и задаётся полем currentTime). Пример настройки `часов`: 

```javascript
var clock = viewer.clock;
clock.startTime = _my_start;
clock.currentTime = _my_start;
clock.stopTime = _my_stop;
``` 
 )
 

----------------------


Привязывание к высотам (учитывание рельефа)
------

Так как корректность движения объектов может быть связана с рельефом (например движение автомобиля 
по дороге), то в Cesium существует возможность корректировать координаты объекта в зависимости от 
его положения относительно рельефа. Достигается это через следующие сущности:
 - 3D-тайлы
 - Обработчик окончания загрузки тайлов
 - Обработчик тактов рендеринга
 - Метод вычисления текущего значения, в соотв. со временем
 - Метод вычисления позиции в соотв. с высотами: `viewer.scene.clampToHeight`
 
Для того чтобы активировать корректировку позиций объекта, при движении которого надо учитывать
рельев, неоходимо после загрузки тайлов привязать функцию корректировки позиции к каждому
такту рендеринга. Разберём это по шагам:
 
 **Загрузка 3D-тайлов**
 
 - Получение ресурса по его идентификатору (точнее его `промиса`, который завершится после
   фактической загрузки ресурса):
   
```javascript
let ionResource = Cesium.IonResource.fromAssetId(40866);
```
   
 - Создание коллекции 3D-тайлов, использую полученный экземпляр `IonResource`:

```javascript
var tileset = new Cesium.Cesium3DTileset({
    url: ionResource
});
```

 - Добаавление коллекции 3D-тайлов к коллекции примитивов сцены

```javascript
viewer.scene.primitives.add(tileset);
```

 - Полностью процесс загрузки 3D-тайлов выглядит следующим образом:

```javascript
let ionResource = Cesium.IonResource.fromAssetId(40866);
var tileset = new Cesium.Cesium3DTileset({
    url: ionResource
});
viewer.scene.primitives.add(tileset);
``` 

**Проверка функционала учитывания высот и привязывание функции корректировки**

 - Проверка возможости скорректировать высоту:
```javascript
if (scene.clampToHeightSupported) { ... }
```

 - Привязывание ф-ии корректировки (если проверка возможности корректировать 
 высоту, прошла успешно):
```javascript
tileset.initialTilesLoaded.addEventListener(start);
/*
tileset - это полученная ранее коллекция 3D-тайлов, т.е. экземпляр класса Cesium.Cesium3DTileset
start - это опредленная ранее (либо определённая впоследствии через `function start() {...}`) 
        функция с кодом, который надо активировать после загрузки тайлов, в частности с кодом
        корректировки позиций объекта в соотв. с высотами
*/
```

**Определение ф-ии корректировки**

 - В функции корректировки необходимо включить анимацию (если она ещё не включена):
```javascript
viewer.clock.shouldAnimate = true;
```
 
 - Определить массив объектов, для которых нужно корректировать их положение с учётом 
 высот (рельефа), т.е. тех, для которых нужно исключить автоматическую смену позиции:
 
```javascript
let objectsToExclude = [entity];
```
 
 - Далее, необходимо для каждого такта рендера определить ф-ию в которой для определённых
 объектов будет осуществляться корректировка позиции с учётом высот:
 
```javascript
scene.postRender.addEventListener(function () {
  ...
});
```

 - Внутри функции корректировки нужно вычислить новую позицию объекта с учётом текущего времени: 
 
```javascript
// Get new position by time (with activated animation)
  let position = positionProperty.getValue(clock.currentTime);
```

 - А также, необходимо поменять позицию объекта(ов) с учётом текущей высоты
 
```javascript
  // Correct position with high
  entity.position = scene.clampToHeight(position, objectsToExclude);
```

 - Полностью определение ф-ии корректировки может выглядеть так:
 
```javascript
function start() {
    clock.shouldAnimate = true;

    let objectsToExclude = [entity];
    scene.postRender.addEventListener(function () {
      // Get new position by time (with activated animation)
      let position = positionProperty.getValue(clock.currentTime);
      // Correct position with high
      entity.position = scene.clampToHeight(position, objectsToExclude);
    });
}
```

**Полный код корректировки позиции с учётом рельефа:**

```javascript
  let ionResource = Cesium.IonResource.fromAssetId(40866);

  var tileset = new Cesium.Cesium3DTileset({
    url: ionResource
  });
  scene.primitives.add(tileset);

  if (scene.clampToHeightSupported) {
    tileset.initialTilesLoaded.addEventListener(start);
  } else {
    window.alert("This browser does not support clampToHeight.");
  }

  function start() {
    clock.shouldAnimate = true;

    let objectsToExclude = [entity];
    scene.postRender.addEventListener(function () {
      // Get new position by time (with activated animation)
      let position = positionProperty.getValue(clock.currentTime);
      // Correct position with high
      entity.position = scene.clampToHeight(position, objectsToExclude);
    });
  }
```
    


