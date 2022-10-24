# Geozon_ws-02
Прошивка выключателя от GEOZON  WS-02 в esphome + доработка подсветки

![photo_5381870553813597690_y](https://user-images.githubusercontent.com/64173457/197458302-5c93b8c3-d5d2-425a-9193-71c6038d486c.jpg)
Двухклавишный выключатель на чипе от TUYA
https://developer.tuya.com/en/docs/iot/wifie3smodule?id=K9605ua1cx9tv
на борту eps8266 и 1мб памяти. В сети куча вариантов по переделке устройств на этой сборке ( в тасмоту, по воздуху и т.д.)
Добавим еще одно описание моддинга. Изначально выключатель предназначен для работы с фирменным приложением от Геозон. Даже не знаю что-то и не пробовал. Наличие чипа от туйа подразумевает работу с любым приложением от Tuya, это было проверено, выключатель успешно добавился.
Далее нужен паяльник, внешнее питание 3,3 вольта и любой уарт переходник к ПК. Вынимаем из выключателя плату с сенсорами ( держится она только на 6-контактном разьеме с блоком реле и питания)
![TYWE3S_pinout](https://user-images.githubusercontent.com/64173457/197459756-a7d23c38-e8ec-40bc-8a35-e56e34e5da49.png)
Подключаемся к указанным ногам модуля, т.е. подключаем уарт, gpio00  припаиваясь прямо к ногам модуля в на плате со стороны где стоят сенсоры. Подаем внешнее питание 3,3в не подключая блок реле, все должно светится, работать...
![photo_5381870553813597689_y](https://user-images.githubusercontent.com/64173457/197462831-92272d35-140f-4ec0-86d4-3d4a9ac7f997.jpg)
На уарт(GPIO03) висит сенсор клавиши и мешает работе порта. нужно убать один резистор ( перемычку) или поднять ногу чипа сенсора, кому как удобнее.

![photo_5381870553813597696_x](https://user-images.githubusercontent.com/64173457/197460361-aeb1cea2-a695-4dad-8ad3-1e39d59b7925.jpg)

Рекомендую сохранить текущую прошивку. Далее прошиваем как обычно замкнув при старте gpio00 на землю
Проверяем, если все удачно восстанавливаем соединение чипа сенсора с GPIO03.
Мне достался выключатель без красной подсветки кнопок в выключенном состоянии. При этом на плате уже стоят двухцветные светодиоды, но не хватает двух резисторов по 1ком.
Восстанавливаем подсветку ( на время припаивания резисторов светопроводящие накладки клавиш лучше отклеить):
![photo_5381870553813597700_x](https://user-images.githubusercontent.com/64173457/197461644-480946bc-513b-4621-b2ca-b7cc199373c7.jpg)
![photo_5381870553813597699_x](https://user-images.githubusercontent.com/64173457/197461661-ac6d4585-9f3a-4b41-9228-0a3b7124bb5a.jpg)

Пины: (sensor1 - gpio03; rele1 - gpio13 ; led1  - gpio14) (sensor2 - gpio05; rele1 - gpio04 ; led1  - gpio01) ( Status - gpio00)
Кофиг может быть таким:
```
binary_sensor:
  - platform: gpio
    pin:
      number: GPIO3 #1 кнопка
      mode: INPUT_PULLUP
      inverted: True
    filters: 
      - delayed_on: 50ms
      - delayed_off: 50ms
      - delayed_on_off: 50ms
    id: button_1
    on_press:
      - light.toggle: light_1
  - platform: gpio
    pin:
      number: GPIO5 #2 кнопка 
      mode: INPUT_PULLUP
      inverted: True
    filters: 
      - delayed_on: 10ms
      - delayed_off: 10ms
      - delayed_on_off: 10ms
    id: button_2
    on_press:
      - light.toggle: light_2
output:
  - platform: gpio
    pin: GPIO13
    id: relay_1
  - platform: gpio
    pin: GPIO04
    id: relay_2
  - id: led1
    platform: gpio
    inverted: true
    pin: GPIO14
  - id: led2
    platform: gpio
    inverted: true
    pin: GPIO01


light:
  - platform: status_led
    name: Status_$board_name
    id: light_s
    internal: true
    pin:
      number: GPIO0
      inverted: true
  - platform: binary
    name: T1_$board_name
    id: light_1
    output: relay_1
    on_turn_on:
      then:
        - light.turn_on: led_1
    on_turn_off:
      then:
        - light.turn_off: led_1
  - platform: binary
    name: T2_$board_name
    id: light_2
    output: relay_2
    on_turn_on:
      then:
        - light.turn_on: led_2
    on_turn_off:
      then:
        - light.turn_off: led_2
  - platform: binary
    id: led_1
    internal: true
    output: led1

  - platform: binary
    id: led_2
    internal: true
    output: led2
```
