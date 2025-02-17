<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <!-- Масштабирование для мобильных устройств -->
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <!-- Отключаем автоматическое распознавание номеров телефонов в iOS -->
  <meta name="format-detection" content="telephone=no">
  <title>Калькулятор стоимости изделия</title>
  <style>
    /* Подключаем шрифт Roboto для современного вида */
    @import url('https://fonts.googleapis.com/css2?family=Roboto:wght@400;500&display=swap');

    body {
      font-family: 'Roboto', sans-serif;
      background: linear-gradient(135deg, #f5f7fa, #c3cfe2);
      margin: 0;
      padding: 20px;
      color: #333;
    }

    h1 {
      text-align: center;
      margin-bottom: 20px;
    }

    .card {
      background: #fff;
      border-radius: 8px;
      box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
      max-width: 900px;
      margin: 0 auto;
      padding: 20px;
    }

    table {
      width: 100%;
      border-collapse: collapse;
      margin: 0 auto;
    }

    th, td {
      padding: 12px 15px;
      text-align: center;
      border-bottom: 1px solid #ddd;
    }

    th {
      background-color: #f0f4f7;
      font-weight: 500;
    }

    input[type="number"] {
      width: 100%;
      padding: 8px;
      border: 1px solid #ccc;
      border-radius: 4px;
      font-size: 1em;
      box-sizing: border-box;
      transition: border-color 0.3s ease;
      /* Сброс стандартного оформления для кроссбраузерности */
      -webkit-appearance: none;
      -moz-appearance: textfield;
      appearance: textfield;
    }

    input[type="number"]:focus {
      border-color: #007bff;
      outline: none;
      box-shadow: 0 0 5px rgba(0, 123, 255, 0.5);
    }

    .input-group {
      display: flex;
      flex-direction: column;
      align-items: flex-start;
      gap: 5px;
    }

    .small-text {
      font-size: 0.85em;
      color: #777;
    }

    @media (max-width: 768px) {
      th, td {
        font-size: 0.9em;
      }
      .input-group input[type="number"] {
        font-size: 0.9em;
      }
    }
  </style>
</head>
<body>
  <h1>Калькулятор стоимости изделия</h1>
  <div class="card">
    <table>
      <thead>
        <tr>
          <th>Наименование</th>
          <th>Параметры / Цена за единицу</th>
          <th>Количество / Площадь</th>
          <th>Итоговая стоимость ($)</th>
        </tr>
      </thead>
      <tbody>
        <!-- Ряд для панели (МДФ 2800×2700 мм) -->
        <tr>
          <td rowspan="2">
            Панель<br>
            <span class="small-text">(МДФ 2800×2700 мм)</span>
          </td>
          <td>
            <div class="input-group">
              <label for="panelWidth">Ширина (мм):</label>
              <!-- Поле пустое, но для расчётов используется резервное значение 2800 -->
              <input type="number" id="panelWidth" value="" inputmode="numeric">
            </div>
            <div class="input-group">
              <label for="panelHeight">Высота (мм):</label>
              <!-- Резервное значение 700 -->
              <input type="number" id="panelHeight" value="" inputmode="numeric">
            </div>
          </td>
          <td>
            Площадь (м²):<br>
            <span id="panelArea">0.00</span>
          </td>
          <td rowspan="2" id="panelCost">0.00</td>
        </tr>
        <tr>
          <td colspan="2">
            <div class="input-group">
              <label for="panelCount">Количество панелей:</label>
              <!-- Резервное значение 1 -->
              <input type="number" id="panelCount" value=""  inputmode="numeric">
            </div>
          </td>
        </tr>
        <!-- Ряд для шпона -->
        <tr>
          <td>
            Шпон (за м²)<br>
            <span class="small-text">(1 лист ≈ 6 м²)</span>
          </td>
          <td>
            <div class="input-group">
              <label for="veneerPrice">Цена за м²:</label>
              <!-- Резервное значение 1 -->
              <input type="number" id="veneerPrice" value="" step="0.01" inputmode="decimal">
            </div>
          </td>
          <td>
            Потребность (м²):<br>
            <span id="veneerArea">0.00</span>
          </td>
          <td id="veneerTotal">0.00</td>
        </tr>
        <!-- Ряд для лака -->
        <tr>
          <td>Лак (200 г на 1 м²)</td>
          <td>
            <div class="input-group">
              <label for="lacquerPrice">Цена за м²:</label>
              <!-- Резервное значение 3 -->
              <input type="number" id="lacquerPrice" value="" step="0.01" inputmode="decimal">
            </div>
          </td>
          <td>
            Площадь (м²):<br>
            <span id="lacquerArea">0.00</span>
          </td>
          <td id="lacquerTotal">0.00</td>
        </tr>
      </tbody>
      <tfoot>
        <tr>
          <td colspan="3"><strong>Общая сумма</strong></td>
          <td id="grandTotal">0.00</td>
        </tr>
      </tfoot>
    </table>
  </div>

  <script>
    // Если поле пустое, для расчётов используется резервное значение,
    // а само поле остаётся пустым.
    function getValidatedValue(id, defaultValue) {
      const input = document.getElementById(id);
      const value = input.value.trim();
      return value === "" ? parseFloat(defaultValue) : parseFloat(value);
    }

    function calculateTotals() {
      const panelWidth  = getValidatedValue('panelWidth', 2800);
      const panelHeight = getValidatedValue('panelHeight', 700);
      const panelArea   = (panelWidth / 1000) * (panelHeight / 1000);
      document.getElementById('panelArea').innerText = panelArea.toFixed(2);

      const panelCount  = getValidatedValue('panelCount', 1);
      const totalArea   = panelArea * panelCount;

      const veneerPrice = getValidatedValue('veneerPrice', 1);
      document.getElementById('veneerArea').innerText = totalArea.toFixed(2);
      const veneerTotal = totalArea * veneerPrice;
      document.getElementById('veneerTotal').innerText = veneerTotal.toFixed(2);

      const lacquerPrice = getValidatedValue('lacquerPrice', 3);
      document.getElementById('lacquerArea').innerText = totalArea.toFixed(2);
      const lacquerTotal = totalArea * lacquerPrice;
      document.getElementById('lacquerTotal').innerText = lacquerTotal.toFixed(2);

      const panelCost = veneerTotal + lacquerTotal;
      document.getElementById('panelCost').innerText = panelCost.toFixed(2);
      document.getElementById('grandTotal').innerText = panelCost.toFixed(2);
    }

    // Обновляем расчёты при каждом вводе данных или при уходе с поля
    document.querySelectorAll('input[type="number"]').forEach(input => {
      input.addEventListener('input', calculateTotals);
      input.addEventListener('blur', calculateTotals);
    });

    window.addEventListener('load', calculateTotals);
  </script>
</body>
</html>
