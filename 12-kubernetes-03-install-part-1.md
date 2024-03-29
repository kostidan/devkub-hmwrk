# Домашнее задание к занятию "12.3 Развертывание кластера на собственных серверах, лекция 1"
> Поработав с персональным кластером, можно заняться проектами. Вам пришла задача подготовить кластер под новый проект.

> ## Задание 1: Описать требования к кластеру
> Сначала проекту необходимо определить требуемые ресурсы. Известно, что проекту нужны база данных, система кеширования, а само приложение состоит из бекенда и фронтенда. Опишите, какие ресурсы нужны, если известно:
> 
> * База данных должна быть отказоустойчивой. Потребляет 4 ГБ ОЗУ в работе, 1 ядро. 3 копии.
> * Кэш должен быть отказоустойчивый. Потребляет 4 ГБ ОЗУ в работе, 1 ядро. 3 копии.
> * Фронтенд обрабатывает внешние запросы быстро, отдавая статику. Потребляет не более 50 МБ ОЗУ на каждый экземпляр, 0.2 ядра. 5 копий.
> * Бекенд потребляет 600 МБ ОЗУ и по 1 ядру на копию. 10 копий.

*1. Сначала сделайте расчет всех необходимых ресурсов.*

| | ОЗУ |	Кол-во ядер | Кол-во копий	| Всего ОЗУ| 	Всего ядер |
| --- | --- | --- | --- | --- | --- |
| База данных	| 4 |	1 |	3 |	12 |	3 |
| Кэш	| 4 |	1 |	3 |	12 |	3 |
| Фронтенд |	0,05 |	0,2 |	5	| 0,25 |	1 |
| Бэкенд |	0,6 |	1 |	10 |	6	| 10 |
| **Итого:** |	 |	 |	 |	**30,25**	| **17** |


*2. Затем прикиньте количество рабочих нод, которые справятся с такой нагрузкой.*

Допустим, 5 рабочих год

*3. Добавьте к полученным цифрам запас, который учитывает выход из строя как минимум одной ноды.*

Если рабочих нод 5, то при выходе из строя 1 ноды, ее нагрузка распределится на 4 оставшихся. Т.е. запас должен быть 25%

*4. Добавьте служебные ресурсы к нодам. Помните, что для разных типов нод требовния к ресурсам разные.*

На каждой рабочей ноде будет по одной Control Plane и Worker

| | ОЗУ | Кол-во ядер |
| --- | --- | --- |
| Control plane | 2 | 2 |
| Worker | 1 | 1 |

*5. Рассчитайте итоговые цифры.*

**ОЗУ:** 1,25*30,25/5 + 2 + 1 = **10,5625** (округлим до 12)

**CPU:** 1.25*17/5 + 2 + 1 = **7.25** (округлим до 8)

*6. В результате должно быть указано количество нод и их параметры.*

Таким образом получаем 5 рабочих нод, по 8 ядер CPU и 12Гб ОЗУ
