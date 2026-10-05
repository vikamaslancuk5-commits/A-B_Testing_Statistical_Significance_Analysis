# Аналіз A/B-тестування та статистичної значущості 

Даний проект присвячений автоматизації аналізу продуктів за допомогою A/B-тестування. У межах проекту проведено розрахунок статистичної значущості метрик воронки конверсії за допомогою Z-тесту для двох пропорцій, а також створено інтерактивний дашборд у Tableau.

---

## Корисні посилання

* Інтерактивний дашборд у Tableau Public:  
  [Переглянути Tableau Dashboard](https://public.tableau.com/app/profile/viktoriia.maslianchuk/viz/ABTestingSignificance/Dashboard1?publish=yes)

* Python Notebook у Google Colab:  
  [Переглянути Python Ноутбук](https://colab.research.google.com/drive/1qb2OWRSkbnsLZy1AWWSMdsNPfY6Y6T0p#scrollTo=7MDRxfUiqZ78)

* Датасет з результатами (Google Drive):  
  [Завантажити results.csv](https://drive.google.com/file/d/1VuNEZ4wAwnweW2aE5rbZvhjAtrysHi4J/view?usp=sharing)
  
---

##  Використовувані технології та інструменти
* **Python** (Pandas, NumPy, Statsmodels) — обробка даних, проведення Z-тесту, розрахунок p-value та z-score.
* **SQL** — первинна агрегація та підготовка тестових груп і подій воронки.
* **Tableau Public** — інтерактивна візуалізація та підсвічування статично значущих метрик.
* **Google Colab** — середовище виконання розрахунків та фінального звіту.

---

##  Візуалізація результатів (Tableau Dashboard)

> *Нижче наведено приклад інтерактивного дашборду з підсвічуванням статистично значущих метрик.*

![Tableau Dashboard Preview](ВСТАВТЕ_ПОСИЛАННЯ_НА_ФОТО_АБО_ШЛЯХ_ДО_ФАЙЛУ_В_РЕПОЗИТОРІЇ) <!-- Наприклад: ./images/dashboard_preview.png -->

---

## SQL Запит (Первинна підготовка даних)

<details>
<summary><b>Натисніть тут, щоб переглянути SQL-запит (підготовка даних)</b></summary>

```sql
WITH sesion_info AS (
    SELECT
        ss.date,
        ss.ga_session_id,
        sp.country,
        sp.device,
        sp.continent,
        sp.channel,
        ab.test,
        ab.test_group
    FROM `data-analytics-mate.DA.ab_test` AS ab
    JOIN `DA.session` AS ss
        ON ab.ga_session_id = ss.ga_session_id
    JOIN `DA.session_params` AS sp
        ON ab.ga_session_id = sp.ga_session_id
),

sesion_with_orders AS (
    SELECT
        sesion_info.date,
        sesion_info.country,
        sesion_info.device,
        sesion_info.continent,
        sesion_info.channel,
        sesion_info.test,
        sesion_info.test_group,
        COUNT(DISTINCT o.ga_session_id) AS sesion_with_orders
    FROM `DA.order` AS o
    JOIN sesion_info
        ON o.ga_session_id = sesion_info.ga_session_id
    GROUP BY
        sesion_info.date,
        sesion_info.country,
        sesion_info.device,
        sesion_info.continent,
        sesion_info.channel,
        sesion_info.test,
        sesion_info.test_group
),

events AS (
    SELECT
        sesion_info.date,
        sesion_info.country,
        sesion_info.device,
        sesion_info.continent,
        sesion_info.channel,
        sesion_info.test,
        sesion_info.test_group,
        ep.event_name,
        COUNT(ep.ga_session_id) AS event_cnt
    FROM `DA.event_params` AS ep
    JOIN sesion_info
        ON ep.ga_session_id = sesion_info.ga_session_id
    GROUP BY
        sesion_info.date,
        sesion_info.country,
        sesion_info.device,
        sesion_info.continent,
        sesion_info.channel,
        sesion_info.test,
        sesion_info.test_group,
        ep.event_name
),

sesion AS (
    SELECT
        sesion_info.date,
        sesion_info.country,
        sesion_info.device,
        sesion_info.continent,
        sesion_info.channel,
        sesion_info.test,
        sesion_info.test_group,
        COUNT(DISTINCT sesion_info.ga_session_id) AS sesion_cnt
    FROM sesion_info
    GROUP BY
        sesion_info.date,
        sesion_info.country,
        sesion_info.device,
        sesion_info.continent,
        sesion_info.channel,
        sesion_info.test,
        sesion_info.test_group
),

accounts AS (
    SELECT
        sesion_info.date,
        sesion_info.country,
        sesion_info.device,
        sesion_info.continent,
        sesion_info.channel,
        sesion_info.test,
        sesion_info.test_group,
        COUNT(DISTINCT ass.ga_session_id) AS new_account_cnt
    FROM `DA.account_session` AS ass
    JOIN sesion_info
        ON ass.ga_session_id = sesion_info.ga_session_id
    GROUP BY
        sesion_info.date,
        sesion_info.country,
        sesion_info.device,
        sesion_info.continent,
        sesion_info.channel,
        sesion_info.test,
        sesion_info.test_group
)

SELECT
    sesion_with_orders.date,
    sesion_with_orders.country,
    sesion_with_orders.device,
    sesion_with_orders.continent,
    sesion_with_orders.channel,
    sesion_with_orders.test,
    sesion_with_orders.test_group,
    'sesion_with_orders' AS event_name,
    sesion_with_orders.sesion_with_orders AS value
FROM sesion_with_orders

UNION ALL

SELECT
    events.date,
    events.country,
    events.device,
    events.continent,
    events.channel,
    events.test,
    events.test_group,
    event_name,
    event_cnt AS value
FROM events

UNION ALL

SELECT
    sesion.date,
    sesion.country,
    sesion.device,
    sesion.continent,
    sesion.channel,
    sesion.test,
    sesion.test_group,
    'sesion' AS event_name,
    sesion_cnt AS value
FROM sesion

UNION ALL

SELECT
    accounts.date,
    accounts.country,
    accounts.device,
    accounts.continent,
    accounts.channel,
    accounts.test,
    accounts.test_group,
    'new account' AS event_name,
    new_account_cnt AS value
FROM accounts;

</details>

---

## Ключові метрики та методологія

У межах проєкту проаналізовано 4 основні метрики воронки конверсії:
1. add_payment_info / session
2. add_shipping_info / session
3. begin_checkout / session
4. new_accounts / session

### Логіка аналізу:
* Для кожної метрики обчислюються абсолютні та відносні зміни конверсії між контрольною (test_group = 1) та тестовою (test_group = 2) групами.
* За допомогою Z-тесту при рівні значущості alpha = 0.05 розраховуються z-stat та p-value.
* Значущість (significant):
  * TRUE — зміна є статистично значущою (p-value < 0.05).
  * FALSE — зміна є статистично незначущою (p-value >= 0.05).

---

## Python Скрипт (Обчислення у циклі)

Нижче наведено фрагмент коду, використаного для ітеративного розрахунку статистичної значущості для кожного тесту та метрики:

```python
from google.colab import drive
import pandas as pd
import numpy as np
from statsmodels.stats.proportion import proportions_ztest

# Очищення назв подій
df['event_name_clean'] = df['event_name'].astype(str).str.strip().str.lower()

metrics = { 
    'add_payment_info / session':  ('add_payment_info', 'session', 'add_payment_info'), 
    'add_shipping_info / session': ('add_shipping_info', 'session', 'add_shipping_info'), 
    'begin_checkout / session':    ('begin_checkout',    'session', 'begin_checkout'), 
    'new_accounts / session':      ('new accounts',      'session', 'new account') 
}

results = []

for test_number in df['test'].unique():
    test_data = df[df['test'] == test_number]
    ctr_data = test_data[test_data['test_group'] == 1] # Контроль
    var_data = test_data[test_data['test_group'] == 2] # Тест

    for metric_name, (num_display, den_display, num_search) in metrics.items():
        den_search = 'session'

        # Тестова група (Group 2)
        denominator_count_test = var_data[var_data['event_name_clean'] == den_search]['value'].sum()
        numerator_count_test = var_data[var_data['event_name_clean'] == num_search]['value'].sum()
        conversion_rate_test = numerator_count_test / denominator_count_test if denominator_count_test > 0 else np.nan

        # Контрольна група (Group 1)
        denominator_count_control = ctr_data[ctr_data['event_name_clean'] == den_search]['value'].sum()
        numerator_count_control = ctr_data[ctr_data['event_name_clean'] == num_search]['value'].sum()
        conversion_rate_control = numerator_count_control / denominator_count_control if denominator_count_control > 0 else np.nan

        # Відносна зміна метрики (%)
        metric_change = (conversion_rate_test - conversion_rate_control) / conversion_rate_control * 100 if conversion_rate_control and conversion_rate_control > 0 else np.nan

        # Z-тест
        if denominator_count_test > 0 and denominator_count_control > 0:
            z_stat, p_value = proportions_ztest(
                count=[numerator_count_test, numerator_count_control],
                nobs=[denominator_count_test, denominator_count_control]
            )
            is_significant = "TRUE" if p_value < 0.05 else "FALSE"
        else:
            z_stat, p_value, is_significant = np.nan, np.nan, "FALSE"

        results.append({
            "test_number": test_number,
            "metric": metric_name,
            "numerator_event": num_display,
            "denominator_event": den_display,
            "numerator_count_test": numerator_count_test,
            "denominator_count_test": denominator_count_test,
            "conversion_rate_test": conversion_rate_test,
            "numerator_count_control": numerator_count_control,
            "denominator_count_control": denominator_count_control,
            "conversion_rate_control": conversion_rate_control,
            "metric_change_%": metric_change,
            "z_stat": z_stat,
            "p_value": p_value,
            "significant": is_significant
        })

df_results = pd.DataFrame(results)
