<!-- omit from toc -->
# Roslaunch and Params

## Содержание

- [Содержание](#содержание)
- [Распространенные практики](#распространенные-практики)
  - [Объединение узлов под одним пространством имен](#объединение-узлов-под-одним-пространством-имен)
  - [Мапирование топиков](#мапирование-топиков)
  - [Подключение других launch-файлов](#подключение-других-launch-файлов)
  - [Создание опций для launch-файлов](#создание-опций-для-launch-файлов)
- [ROS параметры](#ros-параметры)
- [ROS Python параметры](#ros-python-параметры)
- [Управление параметрами](#управление-параметрами)
  - [Сохранение и загрузка параметров](#сохранение-и-загрузка-параметров)
- [Что нужно сделать?](#что-нужно-сделать)
- [Вопросики](#вопросики)
- [С чем познакомились?](#с-чем-познакомились)
- [Полезные ресурсы](#полезные-ресурсы)

Помните, когда мы делали с вами прошлую лабораторную работу, вы сталкивались с тем, что каждый узел нужно запускать в разных терминалах? Но мы с вами запускали по 2-3 узла, а что будет, если нужно одновременно запустить больше таких? Ведь в реальных системах может присутствовать 10, 20 и более узлов, что может вызвать огромную боль при включении/выключении/проверке всех узлов. Звучит как-то не очень. 

Для облегчения жизни придумали специальный формат, основанный на формате `xml`. Суть данного формата в том, что он позволяет настраивать и запускать группы узлов. Это еще можно назвать скриптом запуска.

Для начала, попробуем рассмотреть простой launch-файл (так они называются, угадайте, как должна называться папка, в которой они хранятся). Смотрим 👇	

```xml
<launch>
    <node name="listener" pkg="rospy_tutorials" type="listener.py" output="screen"/>
    <node name="talker" pkg="rospy_tutorials" type="talker.py" output="screen"/>
</launch>
```

>❔ Раньше мы запускали узел `talker`, указывая `rosrun rospy_tutorials talker`. Когда мы пишем на Python, мы создаем скрипты `.py`. Так что оригинально при работе с Python запускаемые файлы будут с расширением `.py`. А в пакете `rospy_tutorials` разработчики просто скопировали файл `talker.py` в `talker`. Можете убедиться сами, у них размер одинаковый.
Основа launch-файла лежит в тэге `<launch>`, он оборачивает весь файл.

Далее вложенные тэги `<node>` задают запуск узлов. В качестве параметров тэгов указываются:

- `name` - имя, которое присваивается узлу в системе ROS (аналог `__name`)
- `pkg` - название пакета, внутри которого лежит узел
- `type` - название файла узла внутри пакета (для Pyhton - py-файлы, для C++ - исполняемые файл, там уж как назовете при компиляции)
- `output` - (необязательный) режим вывода информации, есть варианты `screen` (в консоль) и `log` (по-умолчанию, в лог-файл)

>❔ При запуске launch-файла также запускается мастер (roscore), если он не был запущен ранее.
Таким файлом из примера удобно пользоваться, так как вместо трех консолей потребуется единственная, в которую будет выкладываться вывод всех узлов, у которых `output="screen"`.

Кстати, такой файл уже есть в пакете `rospy_tutorials`, прочитать его можно командой:

```bash
roscat rospy_tutorials talker_listener.launch
```

Утилита для запуска называется `roslaunch` и вот пример запуска такого файла из пакета `rospy_tutorials`:

```bash
roslaunch rospy_tutorials talker_listener.launch
```

Выключение всех узлов из файла производится нажатием Ctrl+C в терминале, в котором запускали launch-файл. При этом система launch проверяет, что все узлы завершились.

>🦾	Напишите launch-файл `my_first.launch` с таким же содержанием и запустите его. Для этого нужно в пакете создать папку `launch` и в ней создать файл с расширением launch. Сделайте небольшую поправочку - измените имена узлов на `sender` и `receiver`. С помощью утилиты `roslaunch`  запустите файл из своего пакета `study_pkg` и убедитесь, что все работает.

## Распространенные практики

А теперь поговорим о наиболее применяемых практиках относительно launch-файлов.

### Объединение узлов под одним пространством имен

Допустим мы хотим запустить узлы в одном пространстве имен, так как они выполняют определенную задачу (являются подсистемой). Можно это сделать красиво с помощью тэга `<group>` и параметра `ns`:

```xml
<launch>
    <group ns="my_namespace">
        <node name="listener" pkg="rospy_tutorials" type="listener.py" output="screen"/>
        <node name="talker" pkg="rospy_tutorials" type="talker.py" output="screen"/>
    </group>
</launch>
```

>🦾	Объедините запускаемые узлы в файле `my_first.launch` в пространство `new_ns`

### Мапирование топиков

Часто неоходимо переименовать (мапировать) топики узлов. Делается это тэгами `<remap>` внутри тэга `<node>` и параметрами `from` и `to`:

```xml
<launch>
    <node name="listener" pkg="rospy_tutorials" type="listener.py" output="screen">
        <remap from="chatter" to="my_topic"/>
    </node>
    <node name="talker" pkg="rospy_tutorials" type="talker.py" output="screen">
        <remap from="chatter" to="my_topic"/>
    </node>
</launch>
```

>🦾	Смапируйте запускаемые узлы в файле `my_first.launch` к топику `new_topic`

### Подключение других launch-файлов

Иногда можно написать много простых launch-файлов и запустить все их с помощью одного launch-файла. Для этого существует тэг `<include>`:

```xml
<launch>
    <include file="$(find study_pkg)/launch/otherfile.launch" />
    <node name="listener" pkg="rospy_tutorials" type="listener.py" output="screen"/>
    <node name="talker" pkg="rospy_tutorials" type="talker.py" output="screen"/>
</launch>
```

Директива `(find study_pkg)` ищет пакет, имя которого передано аргументом (в нашем случае ищется путь до пакета `study_pkg`) и подставляет путь до него в случае удачного нахождения. Таким образом выполняется сначала launch-файл `otherfile.launch`, а затем остальное содержимое. Уровни вложенности launch-файлов не ограничены (насколько я знаю).

>🦾	Напишите launch-файл `another_one.launch` и добавьте его запуск в `my_first.launch` под пространством имен `new_ns`. Launch-файл `another_one.launch` должен запускать узел `listener` из пакета `roscpp_tutorials`, иметь имя `listener_cpp` и смапировать топик `chatter` к `new_topic`.

### Создание опций для launch-файлов

Иногда создание опреленной системы упрощается, если при запуске существует возможность передать опции файлу запуска. Для launch-файлов существует тэг `<arg>`, который добавляет аргументы launch-файлу:

```xml
<launch>
    <arg name="new_topic_name" default="new_chatter" />

    <node name="listener" pkg="rospy_tutorials" type="listener.py" output="screen">
        <remap from="chatter" to="$(arg new_topic_name)"/>
    </node>
    <node name="talker" pkg="rospy_tutorials" type="talker.py" output="screen">
        <remap from="chatter" to="$(arg new_topic_name)"/>
    </node>
</launch>
```

❔ Директива `(arg new_topic_name)` подставляет значение аргумента. При наличии параметра `default` в тэге `<arg>` установка параметра при запуске launch-файла не обязательна. Для задания значения аргумента выполнение roslaunch происходит следующим образом:

```bash
roslaunch rospy_tutorials talker_listener.launch new_topic_name:=my_topic
```

## ROS параметры

Есть еще один аспект, который называется сервер параметров.

✅ **Параметрами** в ROS называются просто данные, которые хранятся под определенными именами и пространствами имен. Как было рассмотрено ранее, запуск узла в пространстве имен меняет конечное имя узла, а также топика. Аналогично с этим, вся работа узла с параметрами (чтение, запись) происходит в том пространстве имен, которому он принадлежит.

> Сервер параметров хранит параметры и привязан к мастеру. Перезапуск мастера приводит к потере всех ранее заданных параметров.

Пора знакомиться с основной утилитой работы с параметрами =)

```bash
rosparam help
```
```
rosparam is a command-line tool for getting, setting, and deleting parameters from the ROS Parameter Server.

Commands:
    rosparam set    set parameter
    rosparam get    get parameter
    rosparam load   load parameters from file
    rosparam dump   dump parameters to file
    rosparam delete delete parameter
    rosparam list   list parameter names
```

Попробуем проверить список параметров в системе

```bash
rosparam list
```
```
/rosdistro
/roslaunch/uris/host_user_vb__35559
/rosversion
/run_id
```

Давайте поработаем с параметром /rosdistro

```bash
rosparam get /rosdistro
```
```
noetic
```
Удивительно, правда? 🙈

А теперь попробуем задать свой параметр и сразу прочитать его

```bash
rosparam set /my_param 'Hello =)'
rosparam set /my_set '{ 'P': 10.0, 'I': 1.0, 'D' : 0.1 }'

rosparam get /my_param
rosparam get /my_set
rosparam get /my_set/P
```
Результат:
```
Hello =)
{D: 0.1, I: 1.0, P: 10.0}
10.0
```

Вроде все логично 🐥 А теперь попробуйте перезапустить ячейку с выводом списка параметров в системе.

Как видно из вывода хелпа, параметрами также можно управлять, удаляя их, также выгружать в файл и загружать из файла.

## ROS Python параметры

Теперь рассмотрим применение параметров внутри узлов.
Дальнейшие примеры можно производить, вызвав в терминале команду `python`, тогда у вас откроется консоль Python и вводить туда по очереди. Или можно написать все в один скрипт с выводом с помощью `print()` или `rospy.loginfo()`. 

Для начала стандартный и знакомый для Python узла код:

```python
import rospy
rospy.init_node('params_study')
```

Ну и начнем рассматривать, что же можно сделать с параметрами в `rospy`? 

Рассмотрим основные типы обращений к параметрам:

```python
distro = rospy.get_param('/rosdistro')
```
Обращение глобально - как видно, в начале стоит `/`. Не обращаем внимания на ns, ищем именно такой путь параметра и никак иначе!

```python
my_set_param = rospy.get_param('my_set')
```
Обращение локально - в начале не стоит `/`. Допустим мы запустили узел, указав `ns:=my_ns`. Тогда вызов данной функции будет пытаться найти парметр по пути - `/my_ns/my_set` 

```python
my_private_param = rospy.get_param('~private_param')
```
Обращение приватно, поиск будет по пути `/params_study/private_param`. Если задат ns - он будет добавлен перед именем узла. Например, `ns:=my_ns -> /my_ns/params_study/private_param`

Теперь на примере, можно установить разные типы параметров и посмотреть, как они будут формироваться. Зададим параметры из узла, локальный, глобальный и приватный. Первый агрумент - название параметра, второй - значение:

```python
rospy.set_param('~ros_priv_param', 'Hi, I am private =)')
rospy.set_param('ros_loc_param', 'Hi, I am local =)')
rospy.set_param('/ros_glob_param', 'Hi, I am global =)')
```

Выведем список после задания параметров (для примера было задано дополнительный ns - `sample_ns`):

```bash
rosparam list
```
И получаем:

```
/sample_ns/params_study/ros_priv_param
/sample_ns/ros_loc_param
/ros_glob_param
```

Все получилось!

> ❔Глобальный не имеет префикса ns. Приватный отличается тем, что в его префиксе присутствует имя узла. Значит он относится конкретно к узлу. Соответственно, можно запустить много одинаковых узлов (например с помощью флага анонимности) и получить такое же количество параметров.

>🦾 А теперь с помощью утилиты `rosparam` проверьте значения заданных параметров `ros_priv_param`, `ros_glob_param`, `ros_loc_param`

Значит так, мы научились получать параметры от сервера параметров, задавать (если их не существовало - создавать, функция все равно одна и та же). Еще один момент. Бывает такое, хотим работать с параметром, а его нету в сервере (причины могут быть разные). В этом случае при запросе происходит это:

```python
not_exist_param = rospy.get_param('i_do_not_exist')
```
И сейчас вы увидите огромную ошибку, которую пугаться не стоит 👹

```
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/opt/ros/melodic/lib/python2.7/dist-packages/rospy/client.py", line 465, in get_param
    return _param_server[param_name] #MasterProxy does all the magic for us
  File "/opt/ros/melodic/lib/python2.7/dist-packages/rospy/msproxy.py", line 123, in __getitem__
    raise KeyError(key)
KeyError: 'i_do_not_exist'
```

В общем, решение этой проблемы достаточно простое: для начала решим вопрос по пути Python - ловим все исключения (ошибки) и после обрабатываем:

```python
try:
    not_exist_param = rospy.get_param('i_do_not_exist')
except:
    not_exist_param = 'Okay, now it`s default time =0'
```

А еще можно задать значение по умолчанию прямо в вызов вторым аргументом:

```python
not_exist_param = rospy.get_param('i_do_not_exist', 'default_value')
```

И все работает (ура)!

Теперь мы умеем и обрабатывать получение значения по-умолчанию. Есть еще функционал из туториала, удаление параметра, проверка существования параметра и получение списка. Зачем это может понадобиться решайте сами.

```python
# Мы этот параметр ставили ранее
param_name_2_delete = '/ros_glob_param'

# Проверим список параметров, только уже через Python
param_list = rospy.get_param_names()
rospy.loginfo(param_list)

# Наличие можно проверить через функционал ROS    
if rospy.has_param(param_name_2_delete):
    rospy.loginfo('[ROSWay] Parameter exist')
else:
    rospy.loginfo('[ROSWay] Parameter not exist')
    
# И с проверкой удаляем его
if rospy.has_param(param_name_2_delete):
    rospy.delete_param(param_name_2_delete)
    
# Еще раз проверим:
if rospy.has_param(param_name_2_delete):
    rospy.loginfo('[ROSWay] Parameter exist')
else:
    rospy.loginfo('[ROSWay] Parameter not exist')
```

Если вывод похож на:
```
[ROSWay] Parameter exist
[ROSWay] Parameter not exist
```
Значит вы справились с этой проблемой и большие молодцы!

## Управление параметрами

Сейчас хочется обратить внимание на приватные параметры с точки зрения практики. Обычно узлы стартуют с помощью launch-файлов, поэтому задаются параметры внутри с помощью тэгов `<param>`. Пример из одного из файлов планера:

```xml
    <param name="base_global_planner" value="global_planner/GlobalPlanner" />
    <param name="planner_frequency" value="1.0" />
    <param name="planner_patience" value="5.0" />
```
Таким образом задаются локальные параметры (с учетом ns).

Еще немного для понимания, пример из драйвера камеры (внутри тэга `<node>` параметры задаются приватными!):

```xml
<node ns="stereo" name="left_camera" pkg="usb_cam" type="usb_cam_node" output="screen" >
    <param name="video_device" value="/dev/video0" />
	<param name="image_width" value="640" />
	<param name="image_height" value="480" />
</node>
```

Здесь с помощью параметров задается путь девайса (конкретный, так как таких может быть много) и размеры выходного изображения.

### Сохранение и загрузка параметров

Так как при закрытии мастера параметры теряются, было бы неплохо узнать, а как сохранять параметры в файл? А как загружать из файла? Такой функционал есть и утилита все та же:

Сохраним все параметры в файл `/tmp/my_dump.yaml` (обычно расширение `.yaml`) начиная с пространства имен `/` - то есть все параметры. Флаг `-v` для визуального контроля:

```bash
rosparam dump -v '/tmp/my_dump.yaml' '/'
```

Теперь загрузим параметры из файла `/tmp/my_dump.yaml`, но в новое пространство имен:

```bash
rosparam load -v '/tmp/my_dump.yaml' '/my_new_ns_just_to_make_it_long_for_control'
```
Выведем список параметров командой:

```bash
rosparam list
```
Наши параметры будут следующие:

```
/my_new_ns_just_to_make_it_long_for_control/ros_glob_param
/my_new_ns_just_to_make_it_long_for_control/rosdistro
/my_new_ns_just_to_make_it_long_for_control/roslaunch/uris/host_user_vb__38669
/my_new_ns_just_to_make_it_long_for_control/rosversion
/my_new_ns_just_to_make_it_long_for_control/run_id
/my_new_ns_just_to_make_it_long_for_control/sample_ns/params_study/ros_priv_param
/my_new_ns_just_to_make_it_long_for_control/sample_ns/ros_loc_param
/ros_glob_param
/rosdistro
/roslaunch/uris/host_user_vb__38669
/rosversion
/run_id
/sample_ns/params_study/ros_priv_param
/sample_ns/ros_loc_param
```

Как видим, у нас появилась полная копия параметров, только в новом пространстве имен. Еще немного практики для понимания пространства имен

Сохраним параметры тольько из `sample_ns`

```bash
rosparam dump -v '/tmp/my_dump_special_ns.yaml' '/sample_ns'
```
Загрузим их в новое пространство:

```bash
rosparam load -v '/tmp/my_dump_special_ns.yaml' '/new_ns_for_special'
```
Посмотрим список наших параметров командой:

```bash
rosparam list
```
И вывод нам даст соответственно следующее:

```
/my_new_ns_just_to_make_it_long_for_control/ros_glob_param
/my_new_ns_just_to_make_it_long_for_control/rosdistro
/my_new_ns_just_to_make_it_long_for_control/roslaunch/uris/host_user_vb__38669
/my_new_ns_just_to_make_it_long_for_control/rosversion
/my_new_ns_just_to_make_it_long_for_control/run_id
/my_new_ns_just_to_make_it_long_for_control/sample_ns/params_study/ros_priv_param
/my_new_ns_just_to_make_it_long_for_control/sample_ns/ros_loc_param
/new_ns_for_special/params_study/ros_priv_param
/new_ns_for_special/ros_loc_param
/ros_glob_param
/rosdistro
/roslaunch/uris/host_user_vb__38669
/rosversion
/run_id
/sample_ns/params_study/ros_priv_param
/sample_ns/ros_loc_param
```

> 🧠 А теперь мозговой штурм! На этом моменте можно очень хорошо понять принцип пространства имен:
На это можно смотреть как на систему папок. Если мы указываем для сохранения конкретное пространство, то все, что лежит внутри ns (далее за `/` этой папки) будет сохранено. При загрузке, мы указываем папку, с которой начать запись. 

Думаем, объяснять функционал `delete` сильно не стоит, поэтому сносим целое пространство и смотрим, что получилось:

```bash
rosparam delete -v '/my_new_ns_just_to_make_it_long_for_control'
```

Далее уже известная нам команда:

```bash
rosparam list
```

```
/new_ns_for_special/params_study/ros_priv_param
/new_ns_for_special/ros_loc_param
/ros_glob_param
/rosdistro
/roslaunch/uris/host_user_vb__38669
/rosversion
/run_id
/sample_ns/params_study/ros_priv_param
/sample_ns/ros_loc_param
```

В итоге мы подчистили наш сервер параметров.

Теперь к практическим навыкам - вспомним, что запуск launch-файла запускает также и мастера, если тот ранее не был запущен. А сервер параметров завязан на мастера. Значит может понадобиться функционал загрузки параметров на момент запуска узлов:

```xml
<rosparam file="config/costmap_common.yaml" command="load" ns="global_costmap" />
```

В этом примере показан тэг `<rosparam>` и его параметры. На самом деле, параметры схожи с опциями утилиты:

- `file` - файл с сохраненными/подгружаемыми параметрами;
- `command` - может быть `[load / dump / delete]`;
- `ns` - пространство имен, куда загрузить / откуда сохранить / что удалить.

> ❔ Формат `<имя параметра> : <значение>` - это специальный формат файлов `YAML`. Для массивов и вложенных параметров происходит обертка вложенности скобками `{}` или просто новой строкой и внутри по идентичному принципу.

```yaml
ros_glob_param: Hi, I am global =)
rosdistro: 'melodic'
roslaunch:
  uris: {host_user_vb__38669: 'http://user-vb:38669/'}
run_id: 1b078410-b789-11e8-91b9-0800278832b1
sample_ns:
  params_study: {ros_priv_param: 'Hi, I am private =)'}
  ros_loc_param: Hi, I am local =)
```

## Что нужно сделать?

Помимо заданий в самом топике, нужно сделать следующее:

>🦾	Мы с вами разобрали launch-файлы и то, как с ними работать. Вот еще одно задание по ним: добавьте аргумент, чтобы можно было задавать новое имя топика в момент запуска launch-файла. Подсказка, как пробросить аргумент через тэг `<include>` есть в полезных ссылках. Таким образом, задаваемое имя топика должно учитываться как в файле `my_first.launch`, так и в файле `another_one.launch`.

> 🦾 Задачка посложнее. Схема прикреплена к задаче:
> - Есть три программы, две из них (Polynominal и Summing) должны запускаться вместе и жить постоянно, обрабатывая запросы.
> - Одна запускается как программа единичного запроса (Request).
> - При запуске Request передается три числа, они через топик идут в узел полинома, там возводятся в степень в зависимости от положения, далее через топик в Summing, возвращается и отдается обратно как ответ.
> <p align="center">
> <img src=../assets/lab3/task.jpg width=500 />
> </p>

## Вопросики

- В чем предназначение `launch` файлов? В чем их главные преимущества?
- Внимание! большой вопрос. Расшифруйте то, что происходит в этом launch-файле

```xml
<launch>
  <arg name="model" default="waffle" doc="model type [burger, waffle, waffle_pi]"/>
  <arg name="x_pos" default="-2.0"/>
  <arg name="y_pos" default="-0.5"/>
  <arg name="z_pos" default="0.0"/>
  
  <param name="model" value="$(arg model)"/>

  <arg name="gz_gui" default="false"/>

  <include file="$(find gazebo_ros)/launch/empty_world.launch">
    <arg name="world_name" value="$(find turtlebot3_gazebo)/worlds/turtlebot3_world.world"/>
    <arg name="paused" value="false"/>
    <arg name="use_sim_time" value="true"/>
    <arg name="gui" value="$(arg gz_gui)"/>
    <arg name="headless" value="false"/>
    <arg name="debug" value="false"/>
  </include>

  <param name="robot_description" command="$(find xacro)/xacro --inorder $(find turtlebot3_description)/urdf/turtlebot3_$(arg model).urdf.xacro" />

  <node pkg="gazebo_ros" type="spawn_model" name="spawn_urdf"  args="-urdf -model turtlebot3_$(arg model) -x $(arg x_pos) -y $(arg y_pos) -z $(arg z_pos) -param robot_description" />

  <node pkg="robot_state_publisher" type="robot_state_publisher" name="robot_state_publisher">
    <param name="publish_frequency" type="double" value="50.0" />
  </node>
</launch>
```

## С чем познакомились?

- Разобрались с утилитой `roslaunch` и рассмотрели ряд тэгов, используемых в формате `XML`.
- Мы познакомились с сервером параметров и утилитой работы с параметрами и научились пользовать параметры в Python
- Рассмотрели практические применения утилит и параметров в `roslaunch`, в том числе познакомились с утилитой `rosparam`
  
## Полезные ресурсы

- [XML](http://wiki.ros.org/roslaunch/XML)
- [Сервер параметров](http://wiki.ros.org/Parameter%20Server)
- [Cтраница из туториала про параметры](http://wiki.ros.org/rospy_tutorials/Tutorials/Parameters)
- [API rospy](http://docs.ros.org/api/rospy/html/)
- [Полезная ссылка](http://wiki.ros.org/roslaunch/XML/include)
