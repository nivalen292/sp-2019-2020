# Увод в курса по системно програмиране
(записки към първото упражнение)

## Преговор

* какво е [*операционна система*](https://en.wikipedia.org/wiki/Operating_system)
* какво е *ядро* ([*kernel*](https://en.wikipedia.org/wiki/Kernel_(operating_system))) на ОС

## Какво означава системно програмиране

* [системното програмиране](https://en.wikipedia.org/wiki/System_programming) има две основни измерения:
  + *kernel development* - разработка на ядра на OC
  + *low-level userspace development* - разработка на програми и библиотеки от ниско ниво, които не са част от ядрото, но отговарят за функционирането, наблюдението и поддръжката на различни компоненти на системата
    - имплементации на стандартната C библиотека
    - имплементации на стандартните библиотеки на други системни езици (например C++ и Rust)
    - подсистеми за управление на услуги (*service managers*) - например [*sysvinit*](https://en.wikipedia.org/wiki/Init), [*systemd*](https://en.wikipedia.org/wiki/Systemd), [*OpenRC*](https://en.wikipedia.org/wiki/OpenRC)
    - подсистеми за конфигуриране на мрежи (*network managers*) - например [*NetworkManager*](https://en.wikipedia.org/wiki/NetworkManager)
    - графични подсистеми - например [*Xorg*](https://en.wikipedia.org/wiki/X.Org_Server), [*Wayland*](https://en.wikipedia.org/wiki/Wayland_(display_server_protocol))
    - звукови подсистеми - например [*PulseAudio*](https://en.wikipedia.org/wiki/PulseAudio)
    - и много други
* това разделение е условно, тъй като според някои дефиниции ядрото е **самата** операционната система и всички останали програми всъщност са приложни такива

## Езици за системно програмиране

* най-популярният език за системно програмиране е [C](https://en.wikipedia.org/wiki/C_(programming_language))
  + на него са реализирани повечето Unix-like ОС, включително и Линукс
* съществуват и операционни системи, написани на C++, като например [*Haiku*](https://en.wikipedia.org/wiki/Haiku_(operating_system))
* има дори и ОС, написани на екзотични езици като Rust, като например [*Redox*](https://en.wikipedia.org/wiki/Redox_(operating_system))

## Системни извиквания ([*system calls*](https://en.wikipedia.org/wiki/System_call) или накратко *syscalls*)

* основният интерфейс на (ядрото на) една ОС
* най-стабилният интерфейс в една ОС
* чрез тях програмата предава контрола върху процесора (CPU) на ядрото на ОС до момента на връщането на резултата от системното извикване
* извършват се чрез специално софтуерно прекъсване ([*software interrupt*](https://en.wikipedia.org/wiki/Interrupt)), например `int 0x80` 
  + в семейството от архитектури *x86* (т.е.*IA-32*, *AMD64* и др.) инструкцията за извършване на софтуерно прекъсване е с мнемоничен код `int` 
  + в Unix-like ОС това прекъсване обикновено е с код `0x80` (в шестнайсетична бройна система)

## Стандартна C библиотека в Unix ([*libc*](https://en.wikipedia.org/wiki/C_standard_library))

* основната библиотека в една Unix-базирана ОС
  + на практика такава библиотека присъства във всяка една съвременна операционна система (включително и Windows), тъй като базовите интерфейси във всички от тях са описани на езика C
    - Unix популяризира използването на C за тези цели, тъй като е първата операционна система, имплементирана на него
* използва се за реализацията на други библиотеки и програми (както системни, така и приложни)
* тя се състои от дефиниции на базовите програмни конструкции, описани в ISO C и POSIX стандартите (функции, типове, макроси и т.н.)
* разделя се условно на две части в зависимост от това от кой стандарт идва съответната програмна конструкция:
  + *ISO C/ANSI C основа*:
    - [*ANSI C*](https://en.wikipedia.org/wiki/ANSI_C) е стандарт, в който са описани дефинициите на всяка една версия на езика C, както и базовите програмни конструкции, които се използват заедно с тях
    - тази част от библиотеката имплементира въпросните програмни конструкции
      * тя се състои основно от реализации на структури от данни и алгоритми
    - примери за функции от *ANSI C* частта са `fopen(3)` , `strlen(3)` , `abs(3)` 
  + *POSIX-специфична част*:
    - [*POSIX*](https://en.wikipedia.org/wiki/POSIX) е стандарт, в който са описани изискванията, които трябва да бъдат изпълнени, за да може да се нарече една ОС POSIX-съвместима (неформалното наименование за което е тя да е Unix-like)
    - в POSIX има раздел, който описва програмните конструкции, използвани в една [*POSIX-съвместима C библиотека*](https://en.wikipedia.org/wiki/C_POSIX_library)
      * този раздел от стандарта включва и дефинициите от *ANSI C* стандарта, като ги разширява и допълва със специфични за него конструкции
      * тази част от библиотеката имплементира въпросните програмни конструкции
* алтернативно C библиотеката може да се раздели на две части в зависимост от ролята на съответните функции:
  + *syscall wrappers* - функции, които дават възможност на C програмата да извършва системни извиквания, като например `open(2)` , `read(2)` , `fork(2)` и т.н.
  + допълнителни библиотечни функции - те също могат да се подразделят на два вида:
    - функции, които реализират определени абстракции над системните извиквания, като например `fopen(3)` , `readdir(3)` , `execlp(3)` и т.н.  Някои от тези функции извършват системни извиквания само в определени ситуации: например `malloc(3)` извиква `sbrk(2)` само ако има нужда от разширяване на размера на *data segment*-а на програмата, за да се събере в него алокираният блок памет
    - функции, които реализират функционалности, които не извършват системни извиквания, като например `memcpy(3)` , `strcmp(3)` , `pow(3)` и т.н.
* всички други библиотеки се базират на нея
  + обикновено C библиотеките за дадената ОС я използват директно
  + стандартните библиотеки за другите езици за програмиране предоставят wrapper-и за конструкциите от стандартната C библиотека, които се ползват от библиотеките и програмите, написани на тези езици
* всички приложни програми зависят от нея (пряко или непряко)

## Режими на изпълнение на една програма (*преговор*)

* системен режим (*kernel mode*) - програмата минава в този режим при извършване на системно извикване до момента на връщането на резултат от това извикване
* потребителски режим (*user mode*) - нормалният режим на изпълнение на една програма:
  + процесорът изпълнява инструкциите от кода на самата програма
  + извикванията на библиотечни функции също се осъществяват в този режим

## Видове ядра на ОС

* [*монолитно ядро*](https://en.wikipedia.org/wiki/Monolithic_kernel):
  + дефинира абстрактен интерфейс от високо ниво над хардуера на системата
  + (почти) цялата функционалност на ОС е имплементирана в ядрото и работи в системен режим:
    - управление на паметта - виртуална памет ([*virtual memory*](https://en.wikipedia.org/wiki/Virtual_memory)), защита на адресното пространство и т.н.
    - управление на задачите - планировчик ([*scheduler*](https://en.wikipedia.org/wiki/Scheduling_(computing))) и т.н.
    - конкурентност и синхронизация на задачите - междупроцесна комуникация ([*IPC-](https://en.wikipedia.org/wiki/Inter-process_communication))
    - файлова система - виртуална файлова система ([*VFS*](https://en.wikipedia.org/wiki/Virtual_file_system)), конкретни имплементации на файлови системи
    - мрежова свързаност - имплементация на TCP/IP стек ([*Internet protocol suite*](https://en.wikipedia.org/wiki/Internet_protocol_suite))
    - драйвери ([*drivers*](https://en.wikipedia.org/wiki/Device_driver))
    - и т.н.
* [*микроядро*](https://en.wikipedia.org/wiki/Microkernel) - реализира абсолютния минимум, който е нужен за функционирането на една ОС:
  + управление на паметта - виртуална памет, защита на адресното пространство
  + управление на задачите - планировчик
  + конкурентност и синхронизация на задачите - междупроцесна комуникация
  + обикновено не се реализират в ядрото неща като драйвери, файлови системи и мрежов стек, а се изпълняват като потребителски програми (т.нар.*сървъри*)
* [*хибридно ядро*](https://en.wikipedia.org/wiki/Hybrid_kernel) - редуциран вариант на монолитното ядро, който реализира по-ограничен обхват от функционалности в сравнение с едно монолитно ядро (подобно на микроядро), но има структура и дефинира интерфейс, подобен на такъв на монолитно ядро

## Какво е *Линукс* (*Linux*)

* Unix-like операционна система
* има монолитно ядро
* ще ползваме него за целта на упражненията
* Linux дистрибуция - това е Линукс ядро, пакетирано заедно със всичкия софтуер, необходим за функционирането на една ОС
  + най-разпространената дистрибуция е [*Ubuntu*](https://en.wikipedia.org/wiki/Ubuntu)
    - препоръчва се да използвате нея за имплементация и тестване на задачите
    - Ubuntu може лесно да се инсталира във [*VirtualBox*](https://www.virtualbox.org/) - дори има [готови изображения с него](https://www.osboxes.org/ubuntu/)

