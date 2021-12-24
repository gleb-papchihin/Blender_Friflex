### Not TLDR

> *« План, что и говорить, был превосходный: простой и ясный, лучше не придумать.    
> Недостаток у него был только один: было совершенно неизвестно, как привести его в исполнение »   
> — Льюис Кэрролл. Приключения Алисы в Стране чудес.*

Вы слышали про генерацию данных? Вероятно, да. Вероятно, вам, как и нам понравилаь эта идея...   
Но как хорошо подметил Льюис Кэрролл, вопросы начинают появляться, когда приходит пора исполнять задуманное.   
Поэтому вместо новогоднего обсуждения плюсов и минусов, попробуем пролить свет на генерацию данных c помощью [Blender](https://www.blender.org/).


### Интерфейс

Сразу после запуска нас встречает необычный интерфейс. Лично у меня начали разбегаться глаза :)   
Через час, понимаешь, что все не так страшно.   
Через два, начинаешь жонглировать разными настройками. В хорошем плане.   


**Консоль**

![Открываем консоль](https://github.com/gleb-papchihin/git_crash/blob/master/console.webm)


**Аддоны (a.k.a. Библиотеки)**

Стоит умомянуть, что библиотеки для blender можно писать на python...   
Если предыдущее утверждение показалось вам очевидным, то вот ещё два:   

- blender загружает аддоны из локального окружения. 
На Linux путь примерно такой: ./blender/2.93/python/lib/python3.9

- Чтобы blender увидел изменения в вашем аддоне, нужно нажать на эту кнопочку:

![Обновляем аддонны](https://github.com/gleb-papchihin/git_crash/blob/master/reload.png)


**Дальнейшее чтение**

Есть ещё много всего интересного!


### Объекты

Немного разобрались с интерфейсом. Самое время, изучить способности Blender API. 


**Импортируем всё необходимое**

``` python
from bpy_extras.object_utils import world_to_camera_view
from dataclasses import dataclass
from mathutils import Vector
from operator import truediv
import bpy
```

**Получение объектов**

``` python
cube = bpy.data.objects['Cube']
collection = bpy.data.collections['Collection']
objects = collection.all_objects()
```

**Удивительные свойства и как их менять**

``` python
cube.location
# >>> Vector((0.0, 0.0, 0.0))

cube.location.xy
# >>> Vector((0.0, 0.0))

cube.location.xy = (0, 0)
cube.location[0:2] = (0, 0)
```

Отдельное место занимает свойство bound_box


``` python
def bounding_box_local(object: bpy.types.Object) -> List[Vector]:
    return list(map(Vector, object.bound_box))

def bounding_box(object: bpy.types.Object) -> List[Vector]:
    return [
        object.matrix_world @ point 
        for point in bounding_box_local(object)
        ]
```

 **Если вы захотели создать новый объект**
 
 ```
 def create(name, collection, object_data = None):
    object = bpy.data.objects.new(name, object_data) 
    collection.objects.link(object)
    return object

def create_camera(name, collection):
    camera = bpy.data.cameras.new(name)
    return create(name, collection, camera)
 ```
