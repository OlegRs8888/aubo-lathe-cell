## Логика управления

### 1. Общие положения

1.1. Управление дверями станка и тисками осуществляется программируемым логическим контроллером Delta DVP12SE11R с 
	 модулями расширения DVP16SP11R и DVP08SP11R.  

1.2. Внешние системы (ЧПУ, робот) обмениваются дискретными сигналами с ПЛК по протоколу Modbus TCP:  
- все дискретные входы ПЛК (X) отображаются в области RX, начиная с адреса 0x0400;  
- все дискретные выходы ПЛК (Y) отображаются в области RY, начиная с адреса 0x0500.  

1.3. Контроллер робота:  
- читает только дискретные входы ПЛК (область RX);  
- не управляет подсветкой кнопок (выходами LED), не записывает в соответствующие RY‑регистры.  

1.4. Режим работы станка определяется по сигналам светофора ЧПУ, заведённым в ПЛК:  
- X_CNC_RED = X0 DVP08SP (красная лампа);  
- X_CNC_YELLOW = X1 DVP08SP (жёлтая лампа);  
- X_CNC_GREEN = X2 DVP08SP (зелёная лампа).  

1.5. Внутренние признаки режима:  
- AUTO_RUN = 1, если X_CNC_GREEN = 1, иначе 0 (станок в автоматическом режиме, выполняется УП);  
- ALARM_STOP = 1, если X_CNC_RED = 1, иначе 0 (аварийная остановка/блокировка).  

---

### 2. Логика управления зонами (выбор зоны)

2.1. Кнопки выбора зон:  
- PB_ZONE_L_EN → X_PBZONE_L = X3 DVP16SP1 (левая зона);  
- PB_ZONE_R_EN → X_PBZONE_R = X4 DVP16SP1 (правая зона).  

2.2. Подсветка выбора зон:  
- Y_LEDZONE_L = Y3 DVP16SP1 (подсветка PB_ZONE_L_EN);  
- Y_LEDZONE_R = Y4 DVP16SP1 (подсветка PB_ZONE_R_EN).  

2.3. Логика работы:  
- при X_PBZONE_L = 1 ПЛК устанавливает Y_LEDZONE_L = 1; при X_PBZONE_L = 0 – Y_LEDZONE_L = 0;  
- при X_PBZONE_R = 1 ПЛК устанавливает Y_LEDZONE_R = 1; при X_PBZONE_R = 0 – Y_LEDZONE_R = 0;  
- сигналы X_PBZONE_L и X_PBZONE_R безусловно передаются роботу по Modbus (RX 0x040B и 0x040C соответственно);  
- управление выходами Y_LEDZONE_L и Y_LEDZONE_R контроллером робота не запрещено, но не предусматривается, так как 
  они являются функцией только выходов ПЛК Delta.  

---

### 3. Логика управления тисками

3.1. Источники команд тисков:  
- контроллер робота (команды CLAMP/UNCLAMP по Modbus на каждую зону);  
- оператор через кнопки на ручном посту (PB_CLAMP_LEFT, PB_CLAMP_RIGHT).  

3.2. Входы ручного управления тисками:  
- PB_CLAMP_LEFT → X_PBCLAMP_L = X5 DVP16SP1 (левая зона);  
- PB_CLAMP_RIGHT → X_PBCLAMP_R = X6 DVP16SP1 (правая зона).  

3.3. Выходы на катушки тисков (через линию FLEXICORE):  
- COILLINE_W (COILLINE_W1…COILLINE_W11) → Y0…Y5 DVP16SP2 – DO на катушки пневмораспределителей зон.  

3.4. Подсветка кнопок управления тисками:  
- Y_LED_CLAMP_L = Y5 DVP16SP1 – подсветка PB_CLAMP_LEFT;  
- Y_LED_CLAMP_R = Y6 DVP16SP1 – подсветка PB_CLAMP_RIGHT.  

3.5. Связь подсветки с катушками тисков:  
- подсветка кнопки PB_CLAMP_LEFT отражает наличие питания на катушке для функции зажима тисков левой зоны:  
  - если катушка для функции зажима тисков левой зоны под напряжением, то Y_LED_CLAMP_L = 1;  
  - если катушка для функции разжима тисков левой зоны под напряжением, то Y_LED_CLAMP_L = 0;  
- аналогично для правой зоны и Y_LED_CLAMP_R;  
- внешние системы (робот) не управляют Y_LED_CLAMP_L и Y_LED_CLAMP_R, они являются функцией только выходов ПЛК.  

3.6. Фильтрация ручных команд по режиму:  

3.6.1. Определение разрешения ручного управления тисками:  
- HandClampEnable = (NOT AUTO_RUN);  
- при AUTO_RUN = 1 (горит зелёная лампа), ручное управление тисками с поста запрещено.  

3.6.2. Формирование операторских команд:  
- CmdClamp_Op_L = X_PBCLAMP_L AND HandClampEnable;  
- CmdClamp_Op_R = X_PBCLAMP_R AND HandClampEnable;  
- при AUTO_RUN = 1 выражения CmdClamp_Op_L и CmdClamp_Op_R принудительно равны 0.  

3.7. Команды робота:  
- CmdClamp_Rob_L / CmdUnclamp_Rob_L – команды зажать/разжать тиски левой зоны от робота;  
- CmdClamp_Rob_R / CmdUnclamp_Rob_R – аналогично для правой зоны;  
- команды робота на тиски обрабатываются без зависимости от AUTO_RUN и не подчиняются общей аварийной логике (ALARM_STOP, ESTOP).  

3.8. Объединение команд:  
- для левой зоны:  
  - CmdClamp_L = CmdClamp_Rob_L OR CmdClamp_Op_L;  
  - CmdUnclamp_L = CmdUnclamp_Rob_L OR CmdUnclamp_Op_L;  
- для правой зоны:  
  - CmdClamp_R = CmdClamp_Rob_R OR CmdClamp_Op_R;  
  - CmdUnclamp_R = CmdUnclamp_Rob_R OR CmdUnclamp_Op_R.  

3.9. Преобразование в выходы:  
- при CmdClamp_L = 1 ПЛК включает соответствующую катушку COILLINE_Wx левой зоны и выполняет функцию зажима;  
- при CmdUnclamp_L = 1 ПЛК выключает катушку зажима и включает/активирует цепь разжатия;  
- аналогично для правой зоны.  

3.10. В аварийном режиме (ALARM_STOP = 1 или ESTOP):  
- ПЛК выполняет никаких действий на сигналами на катушки.  

---

### 4. Логика управления дверями станка

4.1. Входы ручного управления дверями:  
- PB_DOOR_OPEN → X_PBDOOROPEN = X0 DVP16SP1 (кнопка «Открыть дверь»);  
- PB_DOOR_CLOSE → X_PBDOORCLOSE = X1 DVP16SP1 (кнопка «Закрыть дверь»);  
- PB_DOOR_RELAX → X_PBDOORRELAX = X2 DVP16SP1 (кнопка «RELAX» – снять питание со всех катушек дверей).  

4.2. Выходы управления дверями:  
- Y_DOOR_COIL_OPEN / Y_DOOR_COIL_CLOSE – выходы ПЛК на катушки пневмораспределителя дверей (через Y7 VALVEXCOIL и дополнительные DO при необходимости);  
- Y_LED_DOOR_OPEN = Y0 DVP16SP1;  
- Y_LED_DOOR_CLOSE = Y1 DVP16SP1;  
- Y_LED_DOOR_RELAX = Y2 DVP16SP1.  

4.3. Признак доступности ручного управления дверями:  
- DoorHandEnable = (NOT AUTO_RUN) AND (NOT ALARM_STOP);  
- если DoorHandEnable = 1 – ручное управление дверями с поста разрешено;  
- если DoorHandEnable = 0 – ручное управление дверями запрещено.  

4.4. Подсветка кнопок управления дверями:  
- общая индикация доступности ручного управления реализуется через Y_LED_DOOR_OPEN и Y_LED_DOOR_CLOSE, работающие синхронно:  
  - если DoorHandEnable = 1, ПЛК включает Y_LED_DOOR_OPEN и Y_LED_DOOR_CLOSE (подсветка горит, ручной режим доступен);  
  - если DoorHandEnable = 0, ПЛК выключает Y_LED_DOOR_OPEN и Y_LED_DOOR_CLOSE (подсветка не горит, ручной режим недоступен);  
- Y_LED_DOOR_RELAX может использоваться для отдельной индикации режима RELAX по внутренней логике.  

4.5. Команды от робота (двери):  
- робот формирует команды CmdDoorOpen_Rob, CmdDoorClose_Rob, CmdDoorRelax_Rob и передаёт их по Modbus в ПЛК;  
- команды робота управляют Y_DOOR_COIL_OPEN и Y_DOOR_COIL_CLOSE независимо от AUTO_RUN и DoorHandEnable, но всегда 
	подчиняются аварийным условиям (ALARM_STOP, ESTOP).  

4.6. Команды от оператора (двери):  

4.6.1. Формирование операторских команд:  
- CmdDoorOpen_Op = X_PBDOOROPEN AND DoorHandEnable;  
- CmdDoorClose_Op = X_PBDOORCLOSE AND DoorHandEnable;  
- CmdDoorRelax_Op = X_PBDOORRELAX AND DoorHandEnable.  

4.6.2. При AUTO_RUN = 1 (горит зелёная лампа):  
- DoorHandEnable = 0;  
- CmdDoorOpen_Op, CmdDoorClose_Op, CmdDoorRelax_Op = 0 независимо от нажатий;  
- подсветка кнопок управления дверями не горит (Y_LED_DOOR_OPEN = Y_LED_DOOR_CLOSE = 0);  
- сигналы от кнопок не проходят на катушки пневмораспределителей.  

4.6.3. Во всех остальных режимах (зелёная лампа не горит, красная не горит):  
- DoorHandEnable = 1;  
- команды от кнопок «Открыть», «Закрыть» и «RELAX» проходят на логику управления катушками дверей;  
- подсветка кнопок «Открыть/Закрыть» горит, указывая на доступность ручного управления.  

4.7. Объединение команд робота и оператора (двери):  
- CmdDoorOpen = CmdDoorOpen_Rob OR CmdDoorOpen_Op;  
- CmdDoorClose = CmdDoorClose_Rob OR CmdDoorClose_Op;  
- CmdDoorRelax = CmdDoorRelax_Rob OR CmdDoorRelax_Op.  

4.8. Управление катушками дверей:  
- при CmdDoorOpen = 1 ПЛК включает Y_DOOR_COIL_OPEN и выключает Y_DOOR_COIL_CLOSE согласно алгоритму FB дверей;  
- при CmdDoorClose = 1 ПЛК включает Y_DOOR_COIL_CLOSE и выключает Y_DOOR_COIL_OPEN;  
- при CmdDoorRelax = 1 ПЛК выключает Y_DOOR_COIL_OPEN и Y_DOOR_COIL_CLOSE (обе катушки обесточены).  

4.9. Реакция на загорание красной лампы светофора:  
- при ALARM_STOP = 1 (X_CNC_RED = 1) ПЛК немедленно:  
  - обесточивает обе катушки дверей: Y_DOOR_COIL_OPEN = 0, Y_DOOR_COIL_CLOSE = 0;  
  - запрещает ручное управление: DoorHandEnable = 0;  
  - выключает подсветку кнопок дверей: Y_LEDDOOROPEN = Y_LEDDOORCLOSE = 0;  
  - игнорирует команды от кнопок X_PBDOOROPEN/X_PBDOORCLOSE/X_PBDOORRELAX до снятия красной лампы.  

4.10. Приоритеты:  
- приоритет ALARM_STOP (красная лампа) и внешней safety‑цепи (ESTOP) выше любых команд робота и оператора;  
- команды робота по дверям не блокируются автоматическим режимом (AUTO_RUN), но всегда подчиняются аварийной логике.  
