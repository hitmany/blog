---
layout: post
title: "Ленивый перенос образа Linux сервера по сети"
ref: linux-server-perenos
comments: True
lang: ru
tags: linux xming clonezilla
---

Переезд сервера на Linux для меня всегда большое событие. Ежегодно хостеры обновляют линейки выделенных серверов, предлагают сервера мощнее или дешевле. 
Многие мои знакомые для переезда составляют список установленных пакетов и переносят архивы с файлами. Но иногда на сервере много настроек которые когда-то давно были сделаны, но уже забыты.
Мне лень переносить пакеты и настраивать специфичный софт на новом сервере :stuck_out_tongue_winking_eye:

#### Самый ленивый мануал для переезда

Обычно на новом сервере жесткий диск больше или равен старому тарифу. Я просто делаю образ жесткого диска и переношу его на новый.
Задача не копаться в консольных командах и не тратить время на мануалы :laughing: Но задача усложняется, на моем новом сервере Dedibox XC SSD 2016 нет возможности загрузить свой образ ISO rescue диска, хостер дает из списка свои образы. Единственный рабочий образ был на Ubuntu Live CD.

##### Установка дистрибутивов
Загружаем оба сервера с Ubuntu Live CD
Устанавливаем CloneZilla, для этого добавляем репозиторий Clonezilla (там всегда самый свежий релиз)

{% highlight shell %}
$ wget http://drbl.nchc.org.tw/GPG-KEY-DRBL
$ sudo apt-key add GPG-KEY-DRBL
$ sudo nano /etc/apt/sources.list
{% endhighlight %}

Добавляем в конец sources.list   

{% highlight shell %}
deb http://free.nchc.org.tw/ubuntu hardy main restricted universe multiverse
deb http://free.nchc.org.tw/drbl-core drbl stable
{% endhighlight %}

Устанавливаем clonezilla

{% highlight shell %}
$ sudo apt-get update
$ sudo apt-get install clonezilla
{% endhighlight %}

Эти действия нужно выполнить для обоих серверов. 

##### Перенос

Далее переходим к старому серверу и к переносу сервера по SSH, для этого запустим clonezilla в командной строке
   
{% highlight shell %}
$ clonezilla
{% endhighlight %}

Выбираем device-device

![enter image description here]({{ site.baseurl }}/public/images/posts/09042016/clonezilla-device-device.JPG)

Упрощенный режим

![enter image description here]({{ site.baseurl }}/public/images/posts/09042016/clonezilla-beginner.JPG)

Disk to remote disk

![enter image description here]({{ site.baseurl }}/public/images/posts/09042016/clonezilla-disk-to-remote-disk.JPG)

Выбираем диск с которого переносим

![enter image description here]({{ site.baseurl }}/public/images/posts/09042016/clonezilla-choose-disk.JPG)

Я по своему опыту пытался, перенести образ диска, но у меня возникла ошибка **partclone read image_hdr block_size error**
Поэтому перед переносом на старом сервере нужно проверить разделы на ошибки и исправить их.
Выбираем вариант **-fsck-src-part-y**

![enter image description here]({{ site.baseurl }}/public/images/posts/09042016/clonezilla-fsck.JPG)

«Please "Enter" to continue» жмем :blush:<br>
Now we will start to clone data to the target machine. Выбираем «y»

![enter image description here]({{ site.baseurl }}/public/images/posts/09042016/clonezilla-waiting-to-connect.JPG)

Переходим на новый сервер и принимаем образ
{% highlight shell %}
ocs-live-netcfg
{% endhighlight %}
Нужно проверить сеть, если хостер дает IP по DHCP, выбираете его, если статический адрес на live образе уже настроен, то вам остается только выбрать static и нажимать постоянно enter
{% highlight shell %}
ocs-onethefly -s IP_старого_сервера -t sda
{% endhighlight %}
Где sda обозначение вашего диска на новом сервере
Далее нужно два раза согласится «y» с тем что данные будут записаны на новом диске, Ждем окончания копирования

![enter image description here]({{ site.baseurl }}/public/images/posts/09042016/clonezilla-copying.JPG)

Все на этом мы получили полную копию диска.

#### Увеличение размера диска и Xming

У меня на новом сервере диск больше, прочитав мануалы о коносльных fdisk и parted, я испортил раздел с системой. И решил не тратить время и попробовать графический Gparted

На сервере нет графических процессоров и я использую Xming — порт графического сервера для Windows. Все оконные приложения отображаются прямо в Windows.

Качаем [Xming](https://sourceforge.net/projects/xming/files/latest/download?source=files) и устанавливаем, запускаем.
Настраиваем Putty сессию под Xming и переподключаемся

![enter image description here]({{ site.baseurl }}/public/images/posts/09042016/xming-putty.jpg)

Устанавливаем Gparted

{% highlight shell %}
$ sudo apt-get install gparted
{% endhighlight %}

Настраиваем на сервер Xming

{% highlight shell %}
$ DISPLAY=localhost:0
$ export DISPLAY
{% endhighlight %}

В /etc/ssh/sshd_config должен быть установлен **X11Forwarding yes**. Если не установлен то перезапускаем сервис ssh. Получаем красивый интерфейс прямо в Windows, меняем размер диска, это удобно и надежно.

![enter image description here]({{ site.baseurl }}/public/images/posts/09042016/xming-gparted.JPG)

На этом все, не забывайте менять конфигурацию IP. Для этого можно в Live CD примонтировать раздел в котором лежат системные файлы и редактировать конфиг. Мой раздел был sda2

{% highlight shell %}
$ mkdir /mnt/disk2
$ mount /dev/sda2 /mnt/disk2
$ nano /mnt/disk2/etc/network/interfaces
{% endhighlight %}
	
Редактируем, сохраняем и не забываем размонтировать:

{% highlight shell %}
$ umount /dev/sda2
{% endhighlight %}