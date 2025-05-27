# PRiMvVGPS
Результат работы в этом семестре:
**1. Установка Armbian на Orange PI**  
1. Скачали образ Armbian для Orange PI (например, `Armbian_23.02.1_Orangepizero_jammy_current_6.1.11.img.xz`) с сайта armbian.com.  
2. Записали образ на SD-карту через Etcher (или через `dd` в Linux):  
   ```
   sudo dd if=Armbian_23.02.1_Orangepizero_jammy_current_6.1.11.img of=/dev/sdX bs=4M status=progress
   ```  
3. Загрузили Orange PI с SD-карты, выполнили вход (логин: `root`, пароль: `1234`).  
4. Обновили пакеты:  
   ```
   sudo apt update && sudo apt upgrade -y
   ```

---

**2. Установка Python**  
1. Установили Python 3 и зависимости:  
   ```
   sudo apt install python3 python3-pip python3-dev libffi-dev -y
   ```  
2. Добавили виртуальное окружение:  
   ```
   sudo apt install python3-venv -y
   ```

---

**3. Установка Klipper**  
1. Клонировали репозиторий Klipper:  
   ```
   git clone https://github.com/Klipper3d/klipper
   cd klipper
   ```  
2. Создали виртуальное окружение:  
   ```
   python3 -m venv venv
   source venv/bin/activate
   pip install -r scripts/klippy-requirements.txt
   ```  
3. Настроили конфигурацию для Octopus Pro 1.1:  
   ```
   make menuconfig
   ```  
   Выбрали параметры:  
   - Microcontroller Architecture: **STM32**  
   - Processor model: **STM32H743**  
   - Bootloader offset: **32KiB**  
   - Communication interface: **USB** (или UART).  
4. Скомпилировали прошивку:  
   ```
   make
   ```  
   Файл прошивки: `out/klipper.bin`.

---

**4. Прошивка Octopus Pro 1.1**  
1. Перевели плату в режим DFU (замкнули BOOT0 на 3.3V и перезагрузили).  
2. Подключили плату к Orange PI через USB.  
3. Прошили через `dfu-util`:  
   ```
   sudo dfu-util -a 0 -D out/klipper.bin --dfuse-address 0x08000000
   ```  
4. Проверили порт:  
   ```
   ls /dev/serial/by-id/*
   ```

---

**5. Создание конфигурации для принтера**  
1. Создали файл `printer.cfg` в папке `~/klipper_config`:  
   ```
   [mcu]
   serial: /dev/serial/by-id/usb-Klipper_stm32h743xx_123456789-if00

   [printer]
   kinematics: cartesian
   max_velocity: 300
   max_accel: 3000

   [stepper_x]
   step_pin: PE2
   dir_pin: PE3
   enable_pin: !PE4
   microsteps: 16
   rotation_distance: 40
   endstop_pin: ^PC0
   position_endstop: 0
   position_max: 220

   [heater_bed]
   heater_pin: PA1
   sensor_type: ATC Semitec 104GT-2
   sensor_pin: PC3
   min_temp: 0
   max_temp: 130

   [extruder]
   heater_pin: PA2
   sensor_pin: PC4
   min_temp: 0
   max_temp: 300

   [fan]
   pin: PA0
   ```  
2. Запустили Klipper:  
   ```
   ./klippy/klippy.py ~/klipper_config/printer.cfg -l /tmp/klippy.log
   ```

---

**6. Дополнительные шаги (Moonraker + Fluidd)**  
1. Установили Moonraker:  
   ```
   git clone https://github.com/Arksine/moonraker
   cd moonraker
   ./scripts/install-moonraker.sh
   ```  
2. Установили Fluidd:  
   ```
   sudo apt install nginx -y
   git clone https://github.com/cadriel/fluidd
   sudo cp -r fluidd /var/www/html/
   ```  
3. Открыли порт 80:  
   ```
   sudo ufw allow 80/tcp
   ```

---

**7. Проверка работы**  
1. Команды для теста:  
   ```
   FIRMWARE_RESTART
   G28 X
   G1 X100 F3000
   ```  
2. Просмотр логов:  
   ```
   tail -f /tmp/klippy.log
   ```

---

**Итог**:  
- Orange PI работает как хост для Klipper.  
- Octopus Pro 1.1 прошита и настроена.  
- Конфиг принтера адаптирован под параметры железа.  
- Установлен веб-интерфейс Fluidd для управления.
