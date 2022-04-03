1. Создаем виртуалку. Ставим pg
```
sudo apt update
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" |sudo tee  /etc/apt/sources.list.d/pgdg.list
sudo apt update
sudo apt install postgresql-13
```
2. В файле postgresql.conf установим настройки из фалйа
3. Прогоним бенч и получим данные
```
progress: 60.0 s, 486.8 tps, lat 16.402 ms stddev 14.885
progress: 120.0 s, 527.3 tps, lat 15.144 ms stddev 12.686
progress: 180.0 s, 603.0 tps, lat 13.238 ms stddev 11.694
progress: 240.0 s, 576.4 tps, lat 13.850 ms stddev 11.800
progress: 300.0 s, 572.8 tps, lat 13.941 ms stddev 12.016
progress: 360.0 s, 602.4 tps, lat 13.250 ms stddev 11.654
progress: 420.0 s, 586.7 tps, lat 13.609 ms stddev 11.624
progress: 480.0 s, 571.7 tps, lat 13.967 ms stddev 12.120
progress: 540.0 s, 613.1 tps, lat 13.022 ms stddev 10.804
progress: 600.0 s, 618.0 tps, lat 12.918 ms stddev 10.916
progress: 660.0 s, 598.5 tps, lat 13.339 ms stddev 11.528
progress: 720.0 s, 521.6 tps, lat 15.309 ms stddev 13.690
progress: 780.0 s, 482.5 tps, lat 16.553 ms stddev 16.033
progress: 840.0 s, 552.7 tps, lat 14.447 ms stddev 12.353
progress: 900.0 s, 558.2 tps, lat 14.302 ms stddev 12.312
progress: 960.0 s, 527.5 tps, lat 15.143 ms stddev 12.534
progress: 1020.0 s, 518.8 tps, lat 15.392 ms stddev 13.173
progress: 1080.0 s, 526.4 tps, lat 15.172 ms stddev 13.947
progress: 1140.0 s, 598.0 tps, lat 13.352 ms stddev 11.449
progress: 1200.0 s, 581.9 tps, lat 13.717 ms stddev 11.185
progress: 1260.0 s, 448.1 tps, lat 17.828 ms stddev 16.148
progress: 1320.0 s, 560.0 tps, lat 14.259 ms stddev 12.564
progress: 1380.0 s, 509.5 tps, lat 15.676 ms stddev 14.360
progress: 1440.0 s, 516.1 tps, lat 15.473 ms stddev 12.869
progress: 1500.0 s, 527.4 tps, lat 15.141 ms stddev 12.659
progress: 1560.0 s, 592.8 tps, lat 13.470 ms stddev 11.348
progress: 1620.0 s, 579.3 tps, lat 13.781 ms stddev 11.503
progress: 1680.0 s, 565.5 tps, lat 14.122 ms stddev 12.802
progress: 1740.0 s, 566.9 tps, lat 14.083 ms stddev 12.060
progress: 1800.0 s, 535.8 tps, lat 14.906 ms stddev 12.882
progress: 1860.0 s, 538.2 tps, lat 14.836 ms stddev 12.895
progress: 1920.0 s, 550.1 tps, lat 14.514 ms stddev 12.184
progress: 1980.0 s, 566.4 tps, lat 14.097 ms stddev 12.645
progress: 2040.0 s, 581.8 tps, lat 13.724 ms stddev 11.824
progress: 2100.0 s, 526.9 tps, lat 15.157 ms stddev 13.140
progress: 2160.0 s, 566.4 tps, lat 14.095 ms stddev 11.991
progress: 2220.0 s, 593.7 tps, lat 13.447 ms stddev 11.212
progress: 2280.0 s, 573.7 tps, lat 13.918 ms stddev 11.866
progress: 2340.0 s, 547.8 tps, lat 14.572 ms stddev 13.141
progress: 2400.0 s, 463.4 tps, lat 17.241 ms stddev 15.962
progress: 2460.0 s, 539.1 tps, lat 14.814 ms stddev 12.548
progress: 2520.0 s, 444.8 tps, lat 17.956 ms stddev 15.555
progress: 2580.0 s, 476.8 tps, lat 16.751 ms stddev 14.177
progress: 2640.0 s, 482.4 tps, lat 16.553 ms stddev 15.124
progress: 2700.0 s, 503.1 tps, lat 15.874 ms stddev 14.336
progress: 2760.0 s, 522.0 tps, lat 15.299 ms stddev 13.061
progress: 2820.0 s, 523.1 tps, lat 15.265 ms stddev 13.475
progress: 2880.0 s, 553.2 tps, lat 14.434 ms stddev 12.079
progress: 2940.0 s, 539.7 tps, lat 14.797 ms stddev 12.320
progress: 3000.0 s, 613.6 tps, lat 13.011 ms stddev 11.034
progress: 3060.0 s, 579.4 tps, lat 13.780 ms stddev 11.585
progress: 3120.0 s, 609.9 tps, lat 13.088 ms stddev 11.197
progress: 3180.0 s, 432.5 tps, lat 18.470 ms stddev 15.937
progress: 3240.0 s, 574.6 tps, lat 13.896 ms stddev 11.681
progress: 3300.0 s, 566.4 tps, lat 14.096 ms stddev 11.794
progress: 3360.0 s, 596.9 tps, lat 13.377 ms stddev 11.184
progress: 3420.0 s, 559.8 tps, lat 14.264 ms stddev 12.001
progress: 3480.0 s, 498.0 tps, lat 16.038 ms stddev 13.298
progress: 3540.0 s, 554.5 tps, lat 14.401 ms stddev 12.467
progress: 3600.0 s, 553.8 tps, lat 14.421 ms stddev 12.016
```
4. Среднее за последние 10 минут - (579.4+609.9+432.5+574.6+566.4+596.9+559.8+498.0+554.5+553.8)/10 = 552.58 tps
5. После различных конфигураций (
https://postgrespro.ru/docs/postgrespro/9.5/runtime-config-autovacuum) получилось вот так:
```
autovacuum_max_workers=6
autovacuum_naptime='15s'
autovacuum_vacuum_threshold=30
autovacuum_vacuum_scale_factor=0.1
autovacuum_vacuum_cost_delay=10
```

6. Результат: Разброс стал меньше, но производительность упала. 450-460 tps
