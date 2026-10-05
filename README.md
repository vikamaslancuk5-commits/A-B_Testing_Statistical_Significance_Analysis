# Аналіз A/B-тестування та статистичної значущості 

Даний проект присвячений автоматизації аналізу продуктів за допомогою A/B-тестування. У межах проекту проведено розрахунок статистичної значущості метрик воронки конверсії за допомогою Z-тесту для двох пропорцій, а також створено інтерактивний дашборд у Tableau.

---

##  Використовувані технології та інструменти
* **Python** (Pandas, NumPy, Statsmodels) — обробка даних, проведення Z-тесту, розрахунок p-value та z-score.
* **SQL** — первинна агрегація та підготовка тестових груп і подій воронки.
* **Tableau Public** — інтерактивна візуалізація та підсвічування статично значущих метрик.
* **Google Colab** — середовище виконання розрахунків та фінального звіту.

---

## 📈 Посилання на ресурси
* **Інтерактивний дашборд у Tableau Public:** [Переглянути Tableau Dashboard]([ВСТАВТЕ_ПОСИЛАННЯ_НА_ТАБЛО](https://public.tableau.com/app/profile/viktoriia.maslianchuk/viz/ABTestingSignificance/Dashboard1?publish=yes))
* 🐍 **Google Colab Notebook:** [Переглянути Python Ноутбук](ВСТАВТЕ_ПОСИЛАННЯ_НА_COLAB)
* 📁 **CSV з розрахованими результатами:** [Завантажити results.csv](ВСТАВТЕ_ПОСИЛАННЯ_НА_CSV)[cite: 1]

---

## 🖼️ Візуалізація результатів (Tableau Dashboard)

> *Нижче наведено приклад інтерактивного дашборду з підсвічуванням статистично значущих метрик.*

![Tableau Dashboard Preview](ВСТАВТЕ_ПОСИЛАННЯ_НА_ФОТО_АБО_ШЛЯХ_ДО_ФАЙЛУ_В_РЕПОЗИТОРІЇ) <!-- Наприклад: ./images/dashboard_preview.png -->

---

## 💾 SQL Запит (Первинна підготовка даних)

Запит для вигрузки та групування даних за тестовими групами та подіями воронки:

```sql
SELECT 
    test AS test_number,
    test_group,
    event_name,
    COUNT(DISTINCT user_id) AS numerator_count,
    COUNT(DISTINCT CASE WHEN event_name = 'session' THEN user_id END) OVER (PARTITION BY test, test_group) AS denominator_count
FROM 
    ab_test_events
WHERE 
    event_name IN ('session', 'new account', 'begin_checkout', 'add_shipping_info', 'add_payment_info')
GROUP BY 
    test, 
    test_group, 
    event_name;
