---
description:
---

# Анимации

Библиотека поставляется с рядом вспомогательных методов анимации, которые позволяют легко перейти от немедленного обновления свойств к анимированному.

В предыдущей статье о [Shared Values](shared-values.md) мы узнали о хуке `useAnimatedStyle`, который позволяет создать ассоциацию между кодом Reanimated и свойствами представления. Мы также узнали, как выполнять анимированные переходы между общими значениями.

Однако это не единственный способ запуска анимации. Кроме того, Reanimated предоставляет ряд модификаторов анимации и способов ее настройки. В этой статье мы рассмотрим методы, которые можно использовать для выполнения анимированных обновлений представления.

## Анимированные переходы

Один из самых простых способов запустить анимацию в Reanimated - это сделать анимированный переход Shared Value. Анимированное обновление Shared Value требует лишь незначительных изменений по сравнению с немедленным обновлением. Вспомним пример из предыдущей статьи, где мы обновляли Shared Value некоторым случайным числом при каждом нажатии кнопки:

```js
import Animated, {
    useSharedValue,
    useAnimatedStyle,
} from 'react-native-reanimated';

function Box() {
    const offset = useSharedValue(0);

    const animatedStyles = useAnimatedStyle(() => {
        return {
            transform: [{ translateX: offset.value * 255 }],
        };
    });

    return (
        <>
            <Animated.View
                style={[styles.box, animatedStyles]}
            />
            <Button
                onPress={() =>
                    (offset.value = Math.random())
                }
                title="Move"
            />
        </>
    );
}
```

В приведенном выше примере при нажатии на кнопку происходит немедленное обновление общего значения `offset`. Затем значение `offset` сопоставляется с трансляцией представления с помощью `useAnimatedStyle`. В результате, когда мы нажимаем кнопку "Переместить", анимированный бокс перепрыгивает в новое, случайное место, как показано ниже:

![Анимированные переходы](sv-immediate.gif)

С помощью Reanimated такие обновления Shared Value могут быть преобразованы в анимированные обновления путем обертывания целевого значения с помощью одного из помощников анимации, например, [`withTiming`](../api/animations/withTiming.md) или [`withSpring`](../api/animations/withSpring.md). Единственное изменение, которое мы можем сделать сейчас, - это обернуть случайное значение смещения вызовом `withSpring`, как показано ниже:

```js
<Button
    onPress={() => {
        offset.value = withSpring(Math.random());
    }}
    title="Move"
/>
```

Таким образом, вместо присвоения числа Shared Value мы создаем объект анимации, который затем используется для выполнения обновлений Shared Value до тех пор, пока она не достигнет цели.

В результате `offset` Shared Value плавно переходит между текущим и вновь присвоенным случайным числом, что приводит к красивой пружинящей анимации между этими состояниями:

![](sv-spring.gif)

## Анимация в хуке `useAnimatedStyle`

Анимированные переходы Shared Value - не единственный способ инициировать и запускать анимацию. Часто бывает так, что мы хотим анимировать свойства, которые не отображены непосредственно на Shared Value.

Для этого в Reanimated предусмотрена возможность задания анимации непосредственно в хуке [`useAnimatedStyle`](../api/hooks/useAnimatedStyle.md). Для этого можно использовать те же вспомогательные методы анимации из API Reanimated, но вместо того, чтобы использовать их при обновлении Shared Value, можно использовать их для обертывания свойства style value:

```js
const animatedStyles = useAnimatedStyle(() => {
    return {
        transform: [
            {
                translateX: withSpring(offset.value * 255),
            },
        ],
    };
});
```

Как видите, единственным изменением по сравнению с примером из предыдущего раздела является то, что мы обернули значение, передаваемое в `translateX` offset, вызовом `withSpring`. Это сообщает движку Reanimated, что все обновления, сделанные для смещения `translateX`, должны быть анимированы с использованием пружины.

Благодаря этому изменению нам больше не нужно анимировать обновления `offset` Shared Value, и мы можем сохранить "мгновенность" этих обновлений:

```js
<Button
    onPress={() => (offset.value = Math.random())}
    title="Move"
/>
```

В результате мы получим точно такое же поведение, как и при анимации обновления значения `offset`.

Однако в этом случае мы переносим контроль над тем, как должно происходить обновление значений, с места внесения изменений в Shared Value на место определения стилей представления. Такой подход удобнее во многих случаях, особенно когда свойства представления являются производными от Shared Value, а не Shared Value непосредственно отображаются на заданные стили.

Кроме того, хранение всех аспектов стилей представления и переходов в одном месте часто облегчает контроль над кодом компонентов. Это заставляет вас определять все в одном месте, а не разбрасывать по всей кодовой базе, позволяя запускать анимированные переходы из любого места.

## Анимация во встроенных стилях

Для простой анимации в хуке `useAnimatedStyle` без каких-либо вычислений, например, так:

```js
function SomeComponent() {
    const width = useSharedValue(0);
    const animatedStyle = useAnimatedStyle(() => {
        return {
            width: width.value,
        };
    });
    return <Animated.View style={animatedStyle} />;
}
```

Вы можете не использовать `useAnimatedStyle` и использовать встроенные стили, как здесь, чтобы сэкономить на вводе:

```js
function SomeComponent() {
    const width = useSharedValue(0);
    return <Animated.View style={{ width }} />;
}
```

Ширина вьюшки будет меняться при изменении значения `width`.

Обратите внимание, что мы не используем геттер `.value` в инлайн-стилях. Если вы используете `width.value`, то Reanimated использует только текущее значение общей величины. Рассмотрим пример:

```js
width.value = 5;
//...
<Animated.View style={{ width: width.value }} />;
```

эквивалентно:

```js
<Animated.View style={{ width: 5 }} />
```

Передача общего значения без геттера `.value` заставляет Reanimated следить за изменением общего значения.

Чтобы помочь вам избежать ошибки при использовании встроенных стилей, Reanimated выводит предупреждение при использовании геттера `.value` во встроенных стилях. Если вы хотите отключить это предупреждение, установите значение `disableInlineStylesWarning` в `true` в опциях плагина babel в файле `babel.config.js` следующим образом:

```js
module.exports = {
    presets: [
        // ...
    ],
    plugins: [
        // ...
        [
            'react-native-reanimated/plugin',
            { disableInlineStylesWarning: true },
        ],
    ],
};
```

## Прерывание обновлений с анимацией

Анимированные обновления пользовательского интерфейса, по определению, требуют времени для выполнения. Зачастую нежелательно замораживать взаимодействие пользователя с приложением и ждать завершения переходов. Позволяя пользователю взаимодействовать с экраном во время анимации стилевых свойств, фреймворк должен поддерживать возможность прерывания или изменения конфигурации анимации по ходу ее выполнения.

Это относится и к API анимации Reanimated.

Если вы выполняете анимированное обновление общих значений или указываете анимацию в хуке `useAnimatedStyle`, то эта анимация полностью прерывается. В первом случае при обновлении анимируемого Shared Value фреймворк не будет ждать окончания предыдущей анимации, а сразу же инициирует новый переход, начиная с текущей позиции предыдущей анимации. Прерывания также корректно работают для анимаций, определенных в хуке `useAnimatedStyle`. При обновлении стиля и изменении целевого значения для данного свойства по сравнению с последним запуском хука style, новая анимация будет запущена немедленно, начиная с текущей позиции свойства.

Мы считаем, что описанное поведение, когда речь идет о прерываниях, желательно в большинстве случаев использования, и поэтому сделали его по умолчанию. Если же вы хотите подождать со следующей анимацией до завершения предыдущей или отменить текущую анимацию перед началом новой, то в первом случае это можно сделать с помощью обратных вызовов анимации, а во втором - с помощью метода [`cancelAnimation`](../api/animations/cancelAnimation.md).

Чтобы проиллюстрировать работу прерываний на практике, посмотрите на следующее видео, где мы запускаем пример, представленный ранее, но делаем гораздо более частые нажатия на кнопку, чтобы вызвать изменение значения до того, как анимация закончится:

![Прерывание обновлений с анимацией](sv-interruption.gif)

## Настройка анимации

В настоящее время Reanimated предоставляет три встроенных хелпера анимации: [`withTiming`](../api/animations/withTiming.md), [`withSpring`](../api/animations/withSpring.md) и [`withDecay`](../api/animations/withDecay.md).
Существуют способы расширить этот набор за счет собственных анимаций ("хелперы" анимации строятся на основе абстракции [worklets](worklets.md)), но мы пока не готовы документировать это, поскольку планируем некоторые изменения в этой части API.
Однако встроенные методы вместе с модификаторами анимации (о которых мы поговорим позже) уже обеспечивают большую гибкость.
Ниже мы рассмотрим некоторые наиболее распространенные параметры конфигурации помощников анимации, а за полным набором параметров отсылаем к странице документации по методам [`withTiming`](../api/animations/withTiming.md) и [`withSpring`](../api/animations/withSpring.md).

Оба метода-помощника анимации имеют схожую структуру.

В качестве первого параметра они принимают значение target, в качестве второго - объект конфигурации, и, наконец, в качестве последнего параметра - метод обратного вызова.

Начиная с конца, обратный вызов - это метод, который запускается при завершении анимации или при ее прерывании/отмене.
Обратный вызов запускается с единственным аргументом - булевым числом, указывающим, завершилась ли анимация без отмены:

```js
<Button
    onPress={() => {
        offset.value = withSpring(
            Math.random(),
            {},
            (finished) => {
                if (finished) {
                    console.log('ANIMATION ENDED');
                } else {
                    console.log('ANIMATION GOT CANCELLED');
                }
            }
        );
    }}
    title="Move"
/>
```

### Тайминг

Что касается параметров настройки, то они различаются в зависимости от того, какую анимацию мы запускаем.

Для анимации, основанной на времени, мы можем указать длительность и метод смягчения. Параметр `easing` позволяет управлять скоростью продвижения анимации в течение заданного времени.

Вы можете пожелать, чтобы анимация быстро ускорялась, а затем замедлялась к концу, или начиналась медленно, затем ускорялась и снова замедлялась в конце.

Сглаживание может быть описано с помощью [кривой Безье](https://learn.javascript.ru/bezier-curve) благодаря методу `Easing.bezier`, экспортируемому из пакета Reanimated. Но в большинстве случаев достаточно использовать `Easing.in`, `Easing.out` или `Easing.inOut` для настройки кривой тайминга в начале, в конце или в обоих концах соответственно.

По умолчанию длительность анимации тайминга составляет `300` мс, а смягчение по умолчанию представляет собой квадратичную кривую "вход-выход" (`Easing.inOut(Easing.quad)`):

![Тайминг](easeInOutQuad.png)

Вот как запустить анимацию синхронизации с помощью пользовательской конфигурации:

```js
import {
    Easing,
    withTiming,
} from 'react-native-reanimated';

offset.value = withTiming(0, {
    duration: 500,
    easing: Easing.out(Easing.exp),
});
```

Вы можете посетить сайт [easings.net](https://easings.net/ru) и ознакомиться с различными визуализациями easing. Чтобы узнать, как их применять, обратитесь к файлу [Easing.ts](https://github.com/software-mansion/react-native-reanimated/blob/main/src/reanimated2/Easing.ts), в котором определены все вспомогательные методы, связанные со значением easing.

### Spring

В отличие от тайминга, анимация на основе spring не принимает длительность в качестве параметра. Вместо этого время, необходимое для работы пружины, определяется физикой пружины (которая настраивается), начальной скоростью и расстоянием, которое нужно пройти.

Ниже показан пример определения пользовательской анимации пружины и ее сравнение с настройками пружины по умолчанию. Полный список настраиваемых параметров приведен в документации [`withSpring`](../api/animations/withSpring.md).

```js
import Animated, {
    withSpring,
    useAnimatedStyle,
    useSharedValue,
} from 'react-native-reanimated';

function Box() {
    const offset = useSharedValue(0);

    const defaultSpringStyles = useAnimatedStyle(() => {
        return {
            transform: [
                {
                    translateX: withSpring(
                        offset.value * 255
                    ),
                },
            ],
        };
    });

    const customSpringStyles = useAnimatedStyle(() => {
        return {
            transform: [
                {
                    translateX: withSpring(
                        offset.value * 255,
                        {
                            damping: 20,
                            stiffness: 90,
                        }
                    ),
                },
            ],
        };
    });

    return (
        <>
            <Animated.View
                style={[styles.box, defaultSpringStyles]}
            />
            <Animated.View
                style={[styles.box, customSpringStyles]}
            />
            <Button
                onPress={() =>
                    (offset.value = Math.random())
                }
                title="Move"
            />
        </>
    );
}
```

![Spring](twosprings.gif)

В отличие от предыдущего примера, здесь мы определяем анимацию в хуке `useAnimatedStyle`.
Это позволяет использовать одно общее значение, но привязать его к двум стилям представления.

## Модификаторы анимации

Помимо возможности настройки параметров анимации, еще одним способом ее кастомизации является использование модификаторов анимации.
В настоящее время Reanimated предоставляет три модификатора: [`withDelay`](../api/animations/withDelay.md), [`withSequence`](../api/animations/withSequence.md) и [`withRepeat`](../api/animations/withRepeat.md).

Как следует из названия, модификатор `withDelay` заставляет предоставляемую анимацию запускаться с заданной задержкой, модификатор `withSequence` позволяет предоставить несколько анимаций и заставить их запускаться друг за другом. Наконец, модификатор `withRepeat` позволяет повторить предоставленную анимацию несколько раз.

Модификаторы обычно принимают на вход один или несколько объектов анимации с необязательной конфигурацией и возвращают объект, представляющий собой измененную анимацию. Это позволяет обернуть существующие хелперы анимации (или пользовательские хелперы), а также создать цепочку модификаторов, если это необходимо. О том, как можно параметризовать каждый из методов-модификаторов, читайте в документации к ним.

Теперь давайте проверим использование модификаторов на практике и построим анимацию, которая заставляет прямоугольный вид колебаться при нажатии кнопки. Начнем с рендеринга вида и определения параметра вращения Shared Value, который затем используется для запуска анимации:

```js
import Animated, {
    useSharedValue,
    useAnimatedStyle,
} from 'react-native-reanimated';

function WobbleExample(props) {
    const rotation = useSharedValue(0);

    const animatedStyle = useAnimatedStyle(() => {
        return {
            transform: [
                { rotateZ: `${rotation.value}deg` },
            ],
        };
    });

    return (
        <>
            <Animated.View
                style={[styles.box, animatedStyle]}
            />
            <Button
                title="wobble"
                onPress={() => {
                    // will be filled in later
                }}
            />
        </>
    );
}
```

В приведенном выше примере мы создаем переменную Shared Value, которая будет представлять поворот вьюшки. Затем в `useAnimatedStyle` мы сопоставляем эту переменную с атрибутом вращения, добавляя суффикс "deg", чтобы указать, что угол выражается в градусах.

Посмотрим, как теперь можно заставить вращение анимироваться вперед и назад с помощью модификаторов. Вот что мы можем поместить в обработчик `onPress` кнопки:

```js
rotation.value = withRepeat(withTiming(10), 6, true);
```

Приведенный выше код заставит вьюшку выполнить шесть повторений анимации синхронизации между начальным состоянием значения `rotation` (т.е. `0`) и значением `10`.
Третий параметр, передаваемый в метод `withRepeat`, заставляет анимацию запускаться в обратном направлении при каждом втором повторении.
Если установить флаг реверса в значение `true`, то вращение сделает три полных цикла (от `0` до `10` и обратно).
В конце всех шести повторений вращение вернется к нулю.
Вот что произойдет, если мы нажмем на кнопку " wobble":

![Модификаторы анимации](swing.gif)

В приведенном выше коде вращение происходит только в пределах от `0` до `10` градусов.
Для того чтобы вьюшка также поворачивалась влево, мы могли бы начать с `10` и дойти до `10` градусов.
Но мы не можем просто изменить начальное значение на `-10`, так как в этом случае прямоугольник будет перекошен с самого начала.
Один из способов решения этой проблемы - использовать модификатор `withSequence` и, начиная с `0`, сделать первую анимацию до `-10`, затем несколько раз качнуть вид от `-10` до `10`, и, наконец, вернуться от `-10` обратно к `0`.
Вот как будет выглядеть обработчик `onPress`:

```js
rotation.value = withSequence(
    withTiming(-10, { duration: 50 }),
    withRepeat(
        withTiming(ANGLE, { duration: 100 }),
        6,
        true
    ),
    withTiming(0, { duration: 50 })
);
```

В приведенном выше коде мы разместили три анимации в последовательности.

Сначала мы запускаем тайминг до минимального положения качелей (`-10` градусов), после чего запускаем цикл, который проходит между `-10` и `10` градусами шесть раз (как и в предыдущей реализации).

Наконец, мы добавляем завершающую анимацию тайминга, которая заставляет вращение вернуться к нулю. Для окружающей анимации мы передаем длительность, равную половине длительности зацикленной анимации.

Это связано с тем, что эти анимации проходят половину расстояния, поэтому таким образом мы поддерживаем одинаковые скорости для начального, среднего и завершающего взмахов. Ниже представлен конечный результат:

![Модификаторы анимации](wobble.gif)

## Анимация свойств макета

Reanimated позволяет выполнять анимацию, полностью обходясь без основного JavaScript-потока React Native.

Благодаря полной изоляции бегунка анимации логика приложения (рендеринг компонентов, получение и обработка данных и т. д.) не влияет на производительность анимации и позволяет избежать многих непредсказуемых падений кадров.

Разработчики, знакомые с [Animated API](../../../rn/animated.md) и концепцией [Native Driver](https://reactnative.dev/blog/2017/02/14/using-native-driver-for-animated) в React Native, возможно, уже понимают это преимущество, а также знают, что использование Native Driver ограничено некоторым подмножеством свойств представления.

Однако это не так в Reanimated, который поддерживает анимацию **всех** нативных свойств без нагрузки на основной поток JavaScript (включая такие свойства макета, как `width`, `flex` и т.д.). Фактически, это было реализовано еще в первой версии Reanimated, но мы хотели бы еще раз подчеркнуть это, поскольку время от времени получаем вопросы на эту тему.

Однако при обсуждении анимированного обновления свойств макета важно отметить, что, несмотря на то, что мы избегаем обращения к основному потоку JavaScript, некоторые обновления макета могут быть очень дорогими и вызывать значительные задержки, несмотря на то, что выполняются в собственных потоках.

Примером дорогостоящего обновления свойств макета может служить изменение `flex-direction` у контейнера с большим количеством элементов. Такое изменение приведет к тому, что каждый из элементов изменит свое положение, а также изменит свои размеры. Изменение размеров каждого из представлений может привести к дальнейшему пересчету компоновки вложенных представлений вплоть до листовых узлов. Как видите, одно изменение свойства может вызвать множество перерасчетов.

Возможно, когда нам нужно сделать это один раз, все будет работать нормально, но если мы решим выполнять такие вычисления во время анимации для каждого кадра, результат может оказаться неудовлетворительным, особенно на низкопроизводительных устройствах.

## Анимация свойств, не относящихся к стилю

Стили представления, безусловно, являются наиболее часто анимируемыми свойствами. Однако в некоторых случаях важно анимировать и свойства, не относящиеся к стилям.

Это особенно важно, если у нас есть собственные компоненты, которые отображают собственные свойства, которые мы хотим анимировать. В этом случае мы хотим избежать обращений к основному потоку JavaScript для обновления таких свойств во время анимации. К счастью, Reanimated позволяет это сделать, но поскольку свойства не принадлежат стилям, мы не можем просто использовать хук `useAnimatedStyle`.

Для этого Reanimated предоставляет отдельный хук `useAnimatedProps`.

Он работает аналогично `useAnimatedStyle`, но вместо того, чтобы ожидать метод, возвращающий анимированные стили, мы ожидаем, что возвращаемый объект будет содержать свойства, которые мы хотим анимировать. Затем, чтобы подключить анимированные пропсы к представлению, мы предоставляем полученный объект как свойство `animatedProps` для "Анимированной" версии представления, которое мы хотим отобразить (например, `Animated.View`).

Примеры использования хука [`useAnimatedProps`](../api/hooks/useAnimatedProps.md) приведены в документации.

## Общие значения в свойствах

Вы также можете передать разделяемое значение в качестве свойства, как показано ниже:

```js
const AnimatedCircle = Animated.createAnimatedComponent(
    Circle
);

export default function SvgExample() {
    const sv = useSharedValue('50%');

    return (
        <Svg height="200" width="200">
            <AnimatedCircle
                cx="50%"
                cy="50%"
                fill="lime"
                r={sv}
            />
        </Svg>
    );
}
```

Радиус окружности будет меняться в зависимости от разделяемого значения `sv`. Как и в случае с инлайн-стилями, не забывайте передавать общее значение, а не `.value`.

## Ссылки

-   [Animations](https://docs.swmansion.com/react-native-reanimated/docs/fundamentals/animations/)