
Не так давно, мы - IdChess собрали генератор синтетических данных на blender. Во время разработки, заметили, что информации по работе с blender api с точки зрения генерации данных практически нет. Поэтому решили, написать несколько статей, которые объединят полученный опыт и, надеемся, помогут тем, кто решил создать свой собственный синтетический датасет.


# Настройка интерфейса

Первым делом настроим интерфейс: закроем лишние вкладки и откроем «python console».

![интерфейс](https://github.com/gleb-papchihin/git_crash/blob/master/interface.gif)


# Объекты

Доступ ко всем элементам проекта происходит, через модуль bpy.data. Например, получим доступ к объекту «Cube»:

``` python
cube = bpy.data.objects['Cube']
```

bpy.data.objects ведёт себя, как словарь: доступны методы values, keys, items, get и т.д. Хорошо, а что, если мы хотим получить объекты содержащиеся в какой-нибудь коллекции? Просто используем другой атрибут модуля bpy.data:

``` python
objs = bpy.data.collections['Collection'].all_objects
```

Сейчас в переменной objs находятся три объекта: Camera, Cube, Light. Эти объекты являются экземплярами класса bpy.types.Object и имеют общий набор свойств, который позволяет применять разные трансформации.


### location

По умолчанию это свойство отвечает за смещение относительно центра сцены. Чтобы переместить объект, нам нужно изменить свойство. Например, переместим «Cube» куб на координату (1.0, 0.0, 2.0):

``` python
print(cube.location)
# <Vector (0.0000, 0.0000, 0.0000)>

cube.location = (1, 0, 2)

# Эквивалентно

cube.location.xz = (1, 2)

# Эквивалентно

cube.location.x = 1
cube.location.z = 2

# Эквивалентно

cube.location[0] = 1
cube.location[2] = 2

print(cube.location)
# <Vector (1.0000, 0.0000, 2.0000)>
```

![интерфейс](https://github.com/gleb-papchihin/git_crash/blob/master/location.gif)

``` python
from threading import Thread
import time
import math

def apply_simple_animation(cube, period):
    for deg in range(361):
        cube.location.x = 5 * math.sin(math.radians(deg))
        time.sleep(period)

Thread(target = apply_simple_animation, args = (cube, 0.01)).start()
```


### dimensions

Это свойство позволяет растягивать и сжимать объект вдоль осей. Процесс изменения аналогичен location:

``` python
print(cube.dimensions)
# <Vector (2.0000, 2.0000, 2.0000)>

cube.dimensions = (4, 2, 2)

# Эквивалентно

cube.dimensions.x = 4

# Эквивалентно

cube.dimensions[0] = 4

print(cube.dimensions)
# <Vector (4.0000, 2.0000, 2.0000)>
```

Однако, заметим, что для некоторых объектов dimension работать не будет.

``` python
camera = bpy.data.objects['Camera']

print(camera.dimensions)
# <Vector (0.0000, 0.0000, 0.0000)>

camera.dimensions.x = 2

print(camera.dimensions)
# <Vector (0.0000, 0.0000, 0.0000)>
```

Из этого следует, что dimensions будет работать только для объектов, имеющих форму.

![интерфейс](https://github.com/gleb-papchihin/git_crash/blob/master/dimensions.gif)

``` python
from threading import Thread
import time
import math

def apply_simple_animation(cube, period):
    for deg in range(361):
        cube.dimensions.x = 2 + math.sin(math.radians(deg))
        time.sleep(period)

Thread(target = apply_simple_animation, args = (cube, 0.01)).start()
```

### scale

Масштабировать объект можно не только с помощью свойства dimensions, но и scale. Заметим, что изменение одного из этих свойств приведет к изменению другого.

``` python
print(cube.scale)
# <Vector (1.0000, 1.0000, 1.0000)>

cube.scale *= Vector((2, 1, 1))

# Эквиалентно

cube.scale.x *= 2
```

*![интерфейс](https://github.com/gleb-papchihin/git_crash/blob/master/scale.gif)

``` python
from threading import Thread
import time
import math

def apply_simple_animation(cube, period):
    scale = cube.scale.copy()
    for deg in range(361):
        cube.scale = scale * (1 + math.sin(math.radians(deg)))
        time.sleep(period)

Thread(target = apply_simple_animation, args = (cube, 0.01)).start()
```

### rotation_euler

Для вращения объекта используется свойство rotation_euler. Оно немного отличается от location, dimensions и scale:

``` python
print(cube.rotation_euler)
# <Euler (x=0.0000, y=0.0000, z=0.0000), order='XYZ'>

cube.rotation_euler = (
    math.radians(60),
    math.radians(45),
    math.radians(30)
    )

# Эквивалентно

cube.rotation_euler.x = math.radians(60)
cube.rotation_euler.y = math.radians(45)
cube.rotation_euler.z = math.radians(30)

print(cube.rotation_euler)
# <Euler (x=1.0472, y=0.7854, z=0.5236), order='XYZ'>
```

Предыдущий подход перезаписывает значение углов. Но если мы хотим вращать объект, относительно текущего поворота, то для этого можно использовать следующий метод:

``` python
rotation = Euler((
    math.radians(60),
    math.radians(45),
    math.radians(30)
    ))
cube.rotation_euler.rotate(rotation)
```

При вращении объекта с помощью углов Эйлера, нужно помнить об одном из их недостатков - gimble lock. Про это можно прочитать [тут](https://habr.com/ru/post/183116/)

*Гифка*

``` python
from threading import Thread
import time
import math

def apply_simple_animation(cube, period):
    for deg in range(361):
        cube.rotation_euler.x = 2 * math.sin(math.radians(deg))
        cube.rotation_euler.z = 4 * math.sin(math.radians(deg))
        time.sleep(period)

Thread(target = apply_simple_animation, args = (cube, 0.01)).start()
```

### bound_box

Это свойство доступно только для чтения. Оно возвращает 8 точек, описывающих границы объекта. В отличии от location, dimensions и scale, bound_box не использует mathutils.Vector:

``` python
print(type(cube.bound_box))
# <class 'bpy_prop_array'>

print(type(cube.bound_box[0]))
# <class 'bpy_prop_array'>
```

Для удобства преобразуем точки в mathutils.Vector:

``` python
def bounding_box(obj):
    return [Vector(point) for point in obj.bound_box]
```

У этого свойства есть ещё одна небольшая особенность: по умолчания координаты граничных точек расчитываются относительно центра объекта, а не сцены. Чтобы этого избежать, можно использовать следующую функцию:

``` python
def bounding_box_world(obj):
    return [obj.matrix_world @ point for point in bounding_box(obj)]
```

*Гифка: Включаем подсветку bound_box*
