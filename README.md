### Not TLDR

> *« План, что и говорить, был превосходный: простой и ясный, лучше не придумать.    
> Недостаток у него был только один: было совершенно неизвестно, как привести его в исполнение »   
> — Льюис Кэрролл. Приключения Алисы в Стране чудес.*

Вы слышали про генерацию данных? Вероятно, да. Вероятно, вам, как и нам понравилаь эта идея...   
Но как хорошо подметил Льюис Кэрролл, вопросы начинают появляться, когда приходит пора исполнять задуманное.   
Поэтому вместо новогоднего обсуждения плюсов и минусов, попробуем пролить свет на генерацию данных c помощью [Blender](https://www.blender.org/).


### Интерфейс

Сразу после запуска нас встречает необычный интерфейс.   
Лично у меня начали разбегаться глаза :)   
Через час, понимаешь, что все не так страшно.   
Через два, начинаешь жонглировать разными настройками.


**Консоль**

![Открываем консоль](https://github.com/gleb-papchihin/git_crash/blob/master/open.gif)


**Аддоны (a.k.a. Библиотеки)**

Из коробки, blender поступает с настроенным локальным окружением.   
Чтобы приложение увидело библиотеку, достаточно переместить её в   
следующую папку: */blender/2.93/python/lib/python3.9 (Linux).*

Во время разработки, удобно тестировать аддонн через blender-консоль.   
Но стоит учесть, что blender автоматически не обновляет библиотеки    
после их изменения. Ничего страшного. Мы сделаем это вручную!   

![Обновляем аддонны](https://github.com/gleb-papchihin/git_crash/blob/master/reload.png)


### Объекты

Вот мы и добрались до API. В этом блоке научимся обращаться к объектам,   
получать и менять их свойства. Но перед этим представим, что у нас есть    
проект, в котором следующая структура объектов:

```
scene:
    pieces:
        black:
            black.pawn
            black.king
        white:
            white.pawn
            white.king
```

**Импортируем необходимые библиотеки**

``` python
# API
from bpy_extras.object_utils import world_to_camera_view
import bpy

# Библиотек используемые в статье
from dataclasses import dataclass
from mathutils import Vector
from operator import truediv
```

**Получение объектов**

К любому объекту на сцене можно получить доступ через **bpy.data**.   
Посмотрим на наиболее применимые свойства:
1. **bpy.data.objects** - Предоставляет доступ ко всем объектам на сцене.
2. **bpy.data.collections** - Предоставляет доступ ко всем коллекциям.    
Коллекции нужны, когда мы хотим объеденить несколько объектов со схожим   
свойством. К примеру, black_pieces - коллекция, содержащая только чёрные шахматные фигуры.

``` python
white_king = bpy.data.objects['white_king']
white_pieces = bpy.collections['white_pieces'].all_objects()
all_pieces = bpy.collections['collection'].all_objects()
```

**Cвойства объекта**


После того, как мы получили объект, мы можем начать работать с ним.   
Почти все задачи можно решить с помощью следующих свойств:   
1. location
2. dimensions
3. scale
4. rotation_euler
5. bound_box
6. data   

Посмотрим на свойства объекта более детально:

``` python
white_king = bpy.data.objects['white_king']

# location, dimensions, scale

white_king.location
# >>> Vector((0.0, 0.0, 0.0))

white_king.location.xy
# >>> Vector((0.0, 0.0))

# rotation_euler

white_king.rotation_euler
# >>> Euler((0.0, 0.0, 0.0), 'XYZ')
```

Стоит отдельно рассмотреть bound_box:

``` python
def bounding_box_local(object: bpy.types.Object) -> List[Vector]:
    return list(map(Vector, object.bound_box))

def bounding_box(object: bpy.types.Object) -> List[Vector]:
    return [
        object.matrix_world @ point 
        for point in bounding_box_local(object)
        ]
```

**Трансформации**

Большая часть трансформаций происходит через изменение свойств объекта.   
Другими словами, если хотим передвинуть объект, то нужно изменить свойство **location**.   

``` python
white_king = bpy.data.objects['white_king']

white_king.location
# >>> Vector((0.0, 0.0, 0.0))

white_king.location.xy
# >>> Vector((0.0, 0.0))

white_king.location.xy = (0, 0)
white_king.location[0:2] = (0, 0)

```

 **Если вы захотели создать новый объект**
 
 ``` python
 def create(name, collection, object_data = None):
    object = bpy.data.objects.new(name, object_data) 
    collection.objects.link(object)
    return object

def create_camera(name, collection):
    camera = bpy.data.cameras.new(name)
    return create(name, collection, camera)
 ```
