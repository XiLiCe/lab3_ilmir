<h1 alighn="center">Разработка инструмента для анализа данных о потреблении энергии с целью оптимизации использования ресурсов</h1>
<p>Этот репозиторий включает в себя решение соревнования Kaggle, <a href="https://www.kaggle.com/competitions/electricity-consumption/overview">Electricity consumption</a></p>


<a href="#download">Загрузка кода</a><p></p>
<a href="#start_code">Запуск кода</a><p></p>
<a href="#what">Как работает?</a><p></p>
<a href="#konec">Соревнование Kaggle</a>


<h2 id="download">Загружаем код</h2>
<p>Файл lab3.ipynb содержит в себе код моей лаб. работы</p>
<p>Файл train.csv тренировочный набор данных</p>
<p>Файл sample.csv тестовый набор данных</p>
<p>Файл ilmir.csv прогноз моей модели</p>


<h2 id="start_code">Запуск кода</h2>
<p>Для запуска кода я использовал GoogleCollab. После того как вы его открыли нужно загрузить туда файлы sample.csv и train.csv</p><p></p>
<img src="https://sun9-70.userapi.com/impg/qk8ou6bon3bcJ1KZs8tCRfvakWWyDJhvLZlyQQ/xDtcCg4-MxI.jpg?size=260x261&quality=96&sign=aadc0d004bac49456585b66c42f53e98&type=album">
<p>После того как мы всё это сделали, нужно запустить каждый блок кода по порядку</p>
<sup>Может занять длительное время</sup>
<p>После выполнения всего кода, у нас появится файл с названием itog.csv, который уже можно отправлять на соревнования в Kaggle</p>
<img src="https://sun21-2.userapi.com/impg/oo8unzwcsGDMHgLWw5KgojfIR1A0cEDgkwPXCw/lr6PYP1MBKE.jpg?size=260x306&quality=96&sign=5d5955bf54d0b8de2da11d278f6bbe03&type=album">

<h2 id="what">Как работает?</h2>


<h3>Работа с данными и импорт библиотек</h3>
<p>Для начала мы импортируем нужные нам библиотеки, а так же загружаем данные. После чего заменяем значения NaN на 0, так же преобразуем столбец datetime в объекты datetime в формате день-месяц-год часы-минуты-секунды</p><p></p>

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from sklearn.ensemble import RandomForestRegressor
from sklearn.metrics import mean_squared_error, mean_absolute_error
from sklearn.model_selection import GridSearchCV
```
```python
df = pd.read_csv("train.csv")
df.replace(np.nan, 0, inplace=True)
df['datetime'] = pd.to_datetime(df['datetime'], format="%d.%m.%Y %H:%M:%S")
```

<p>После всего этого я создал столбцы hour, day_of_week, day_of_month, month, year. Затем удалил строки содержащие пропущенные ззначения</p><p></p>

```python
df['hour'] = df['datetime'].dt.hour
df['day_of_week'] = df['datetime'].dt.dayofweek
df['day_of_month'] = df['datetime'].dt.day
df['month'] = df['datetime'].dt.month
df['year'] = df['datetime'].dt.year
df = df.dropna()
```

<p>Так же я удалил строки в нашем датасете, в которых значение total было равно нулю.</p><p></p>

```python
df=df[df['total']>0]
```

<p>После, я разделил наш датасет на train и test</p><p></p>

```python
train_size = int(len(df) * 0.8)
train, test = df[:train_size], df[train_size:]
```

<!--<img src="https://sun9-47.userapi.com/impg/hH7zh2R-BErwgO6bvsLkgpg2iFxqErWrck4oWg/UVvjKekfdbk.jpg?size=1821x573&quality=96&sign=a2601631926a2ba2772ab8fd10ec6c97&type=album">-->



<h3>Модель</h3>
<p>Я использовал модель RandomForestRegressor</p><p></p>
<p>Я подготовил переменные для использования в обучении модели, и начал искать оптимальные гиперпараметры</p><p></p>

```python
features = ['hour', 'day_of_week', 'day_of_month', 'month', 'year']
target = 'total'

param_grid = {
    'n_estimators': [300, 600, 1200],
    'min_samples_split': [2, 3, 4],
    'min_samples_leaf': [1, 2, 4]
}
```

```python
model = RandomForestRegressor(random_state=42)
grid_search = GridSearchCV(model, param_grid, cv=3, scoring='r2', verbose=2)
grid_search.fit(train[features], train[target])
```

<!--<img src="https://sun9-17.userapi.com/impg/TPIdqjAoKFq6YYzodjzmEZrY8uoboxIorWrsjA/KctNaxkTP8U.jpg?size=1819x288&quality=96&sign=2a3d03444449f4493b0478962699ba20&type=album">-->


<p>Затем вывел их на экран, после чего начал обучать модель используя оптимальные гиперпараметры</p><p></p>

```python
best_params = grid_search.best_params_
print(f'Наилучшие гиперпараметры: {best_params}')
```
```python
best_model = RandomForestRegressor(**best_params, random_state=42)
best_model.fit(train[features], train[target])
```

<!--<img src="https://sun21-2.userapi.com/impg/CJehV3lAAnOxDG6ISjmiJh9vwNmKo3U_eVd-NQ/owYigzsiO-o.jpg?size=1822x189&quality=96&sign=9e38dfadf3e6ab5f23e4dfb17dfd7178&type=album">-->

<p>Использовал обученную модель что бы сделать прогноз на нашей тестовой выборке. Создал график на котором отображаются фактические и предсказанные значения</p><p></p>

```python
predicted_values = best_model.predict(test[features])
```
```python
plt.figure(figsize=(300, 6))
plt.plot(test['datetime'], test[target], label='Фактические данные', linestyle='-', marker='o')
plt.plot(test['datetime'], predicted_values, label='Предсказанные данные (Random Forest)', linestyle='--', marker='x')
plt.title('Предсказание потребления электроэнергии (Random Forest)')
plt.xlabel('Дата')
plt.ylabel('Потребление электроэнергии')
plt.legend()
plt.show()
```

<!--<img src="https://sun21-2.userapi.com/impg/knwPu6sl3Lzbi1TiT7g8vLrmcOfhmF3nDw8jlg/OVL7e-xjlXw.jpg?size=1812x288&quality=96&sign=f3329be81c5c47af520e00ece6b2c093&type=album">-->
<p>График</p>
<img src="https://github.com/XiLiCe/lab3_ilmir/blob/XiLiCe-patch-1/graf.png?raw=true">



<h3>Работа с Sample</h3>
<p>Я загрузил данные и добавил в него те же столбцы что и в train, заменил NaN на нули</p><p></p>


```python
sample = pd.read_csv("sample.csv")
sample
```
```python
sample['datetime'] = pd.to_datetime(sample['datetime'], format="%d.%m.%Y %H:%M:%S")
sample['hour'] = sample['datetime'].dt.hour
sample['day_of_week'] = sample['datetime'].dt.dayofweek
sample['day_of_month'] = sample['datetime'].dt.day
sample['month'] = sample['datetime'].dt.month
sample['year'] = sample['datetime'].dt.year
```
```python
sample.replace(np.nan, 0, inplace=True)
```

<p>Я использовал обученую модель, чтобы сделать прогнозы для данного набора данных.</p><p></p>

```python
forecasted_values_sample = best_model.predict(sample[features])
```

<p>Добавил в датасет Sample столбец total и заполнил его значениями прогнозов, полученных с использованием модели</p><p></p>

```python
sample['total'] = forecasted_values_sample
```

<!--<img src="https://sun9-75.userapi.com/impg/OhSlfp0joJ_F8NkOamd2XT4-YN_GxBJVUlqf5w/51XLQkEzu0A.jpg?size=1822x417&quality=96&sign=e4aca19b5cfb1ad27f7d7b7795cbd3d0&type=album">-->


<h3>Вывод данных</h3>
<p>создал новый файл с именем result, который содержит только столбцы datetime и total из Sample</p><p></p>

```python
result = sample[['datetime', 'total']]
result
```

<p>Сохранил файл result в CSV формате с именем itog.csv</p>

```python
result.to_csv('itog.csv', index=False, date_format='%d.%m.%Y %H:%M:%S')
```

<!--<img src="https://sun21-1.userapi.com/impg/3AJvccRP3r4G4P2UvOsO77Yj70q9xYLbLZnGSw/MR073mPAmfA.jpg?size=1468x550&quality=96&sign=473a40e18ab78c478845f21fa68834dd&type=album">-->


<h3>Оценка модели</h3>
<p>Я использовал оценку r2, так как она используется в нашем соревновании Kaggle. По итогу получил значение R-squared: 0.8932329258065999</p>

```python
from sklearn.metrics import r2_score

r_squared = r2_score(test['total'], predicted_values)

print(f'R-squared: {r_squared}')
```

<!--<img src="https://sun9-67.userapi.com/impg/-WPaP-y1iI2CZAwZZWYA7RnhTarS2DUiWqS-Ng/7HjkR5aQruQ.jpg?size=736x181&quality=96&sign=b9cf7dc4d249723b33b5c90b07ab3e24&type=album">-->

<h2 id="konec">Соревнование</h2>
<p>Я загрузил своё предсказание в соревнование, и получил там 0.79549 очков. Заняв 31 место.</p>
<img src="https://sun21-1.userapi.com/impg/scE0XMVKZNZ6Rtw5IMBE3pd_ebioIsmgMl9oFA/lFLxZfw4ons.jpg?size=1223x147&quality=96&sign=a60720a32652419c97a65efc8ce90983&type=album">





