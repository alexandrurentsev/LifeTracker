```dataviewjs
// Конфигурация

const PROPERTY_FIELDS = ["sport", "Mood", "cognitive", "sleep"]; // Поля свойств для расчёта среднего
const VAULT_NAME = "obsidian-tracker"; // Название хранилища Obsidian
const NOTES_FOLDER = "Daily"; // Папка, где хранятся заметки
const CELL_SIZE = 15; // Размер ячеек календаря (в пикселях)
const GAP_BETWEEN_MONTHS = 10; // Промежутки между месяцами (в пикселях)
const FONT_SIZE = 9; // Размер шрифта внутри ячеек (в пикселях)
const MONTH_TITLE_FONT_SIZE = 12; // Размер шрифта названия месяца (в пикселях)

// Цветовые настройки
const HUE_RED = 13; // Красный цвет (низкая интенсивность)
const HUE_GREEN = 132; // Зелёный цвет (высокая интенсивность)

const INTENSITY_COLORS = [
    `hsl(${HUE_RED}, 100%, 37%)`,     // Темно-красный (1)
    `hsl(${HUE_RED}, 100%, 50%)`,     // Средний красный (2)
    `hsl(${HUE_RED}, 100%, 60%)`,     // Светло-красный (3)
    `hsl(${HUE_RED}, 100%, 77%)`,     // Очень светлый красный (4)
    `hsl(0, 0%, 80%)`,                // Нейтральный серый (5)
    `hsl(${HUE_GREEN * 0.7}, 70%, 72%)`, // Светло-зелёный (6)
    `hsl(${HUE_GREEN * 0.85}, 43%, 56%)`, // Средний зелёный (7)
    `hsl(${HUE_GREEN}, 49%, 36%)`,    // Темно-зелёный (8)
    `hsl(${HUE_GREEN}, 59%, 24%)`,    // Очень тёмно-зелёный (9)
];

// Основные данные для рендера календаря
const calendarData = { 
    entries: [] // Сюда собираются данные из заметок
};

// Функция для расчёта среднего значения свойств
function calculateAverage(properties, page) {
    let total = 0;
    let count = 0;

    properties.forEach(field => {
        const value = parseFloat(page[field]);
        if (!isNaN(value)) {
            total += value;
            count++;
        }
    });

    return count > 0 ? total / count : null; // Возвращаем null, если нет данных
}

// Сбор данных из заметок
for (let page of dv.pages(`"${NOTES_FOLDER}"`)) {
    const average = calculateAverage(PROPERTY_FIELDS, page);
    if (average === null) continue; // Пропускаем записи без данных

    const intensityIndex = Math.min(Math.max(Math.round(average), 1), 9); // Приводим к диапазону 1-9

    calendarData.entries.push({
        date: page.file.name, // Имя файла используется как дата
        intensity: intensityIndex, // Индекс интенсивности
        color: INTENSITY_COLORS[intensityIndex - 1], // Цвет для уровня интенсивности
    });
}

// Функция для рендера одного месяца
function renderMonthlyCalendar(container, month, year, entries) {
    const monthContainer = document.createElement("div"); // Контейнер для месяца
    monthContainer.style.marginBottom = `${GAP_BETWEEN_MONTHS}px`;

    // Название месяца
    const monthTitle = document.createElement("h4");
    monthTitle.style.fontSize = `${MONTH_TITLE_FONT_SIZE}px`;
    monthTitle.style.textAlign = "center";
    monthTitle.style.marginBottom = "5px";
    monthTitle.textContent = new Date(year, month, 1).toLocaleString("default", { month: "long" });
    monthContainer.appendChild(monthTitle);

    // Сетка дней месяца
    const calendarGrid = document.createElement("div");
    calendarGrid.style.display = "grid";
    calendarGrid.style.gridTemplateColumns = `repeat(7, ${CELL_SIZE}px)`; // 7 колонок
    calendarGrid.style.gap = "1px";
    calendarGrid.style.justifyContent = "center";

    const firstDayOfMonth = new Date(year, month, 1);
    const lastDayOfMonth = new Date(year, month + 1, 0);
    const startDayOfWeek = firstDayOfMonth.getDay();

    // Пустые ячейки перед началом месяца
    for (let i = 0; i < startDayOfWeek; i++) {
        const emptyCell = document.createElement("div");
        emptyCell.style.width = `${CELL_SIZE}px`;
        emptyCell.style.height = `${CELL_SIZE}px`;
        calendarGrid.appendChild(emptyCell);
    }

    // Дни месяца
    for (let day = 1; day <= lastDayOfMonth.getDate(); day++) {
        const dateKey = `${year}-${(month + 1).toString().padStart(2, "0")}-${day.toString().padStart(2, "0")}`;
        const entry = entries.find(e => e.date === dateKey);

        const cell = document.createElement("div");
        cell.style.width = `${CELL_SIZE}px`;
        cell.style.height = `${CELL_SIZE}px`;
        cell.style.borderRadius = "3px";
        cell.style.fontSize = `${FONT_SIZE}px`;
        cell.style.display = "flex";
        cell.style.alignItems = "center";
        cell.style.justifyContent = "center";

        if (entry) {
            const link = document.createElement("a");
            link.href = `obsidian://open?vault=${encodeURIComponent(VAULT_NAME)}&file=${encodeURIComponent(NOTES_FOLDER + "/" + entry.date)}`;
            link.style.textDecoration = "none";
            link.style.color = "white";
            link.textContent = day;

            cell.style.backgroundColor = entry.color;
            cell.appendChild(link);
        } else {
            cell.style.backgroundColor = "lightgray";
            cell.style.color = "black";
            cell.textContent = day;
        }

        calendarGrid.appendChild(cell);
    }

    monthContainer.appendChild(calendarGrid);
    container.appendChild(monthContainer);
}

// Основной код для отображения года
try {
    const today = new Date();
    const year = today.getFullYear();
    const yearContainer = document.createElement("div");

    yearContainer.style.display = "grid";
    yearContainer.style.gridTemplateColumns = "repeat(3, auto)";
    yearContainer.style.gap = "10px";

    for (let month = 0; month < 12; month++) {
        renderMonthlyCalendar(yearContainer, month, year, calendarData.entries);
    }

    this.container.appendChild(yearContainer);
} catch (error) {
    console.error("Ошибка при рендеринге календаря", error);
}

```





```dataviewjs
// Конфигурация для настроек
const CONFIG = {
    PROPERTY_FIELDS: ["Mood", "sport", "cognitive", "sleep"], // Поля для анализа
    NOTES_FOLDER: "Daily", // Папка с заметками

    LABELS: ["Отличное", "Хорошее", "Среднее", "Плохое"], // Категории
    COLORS: ["#4caf50", "#8bc34a", "#ffc107", "#f44336"], // Цвета для категорий
    CHART_DIMENSIONS: { width: 400, height: 400 }, // Размер диаграммы
    CHART_RADIUS: 120 // Радиус круговой диаграммы
};

// Сбор данных из заметок
const categoryData = [0, 0, 0, 0]; // Для категорий: Отличное, Хорошее, Среднее, Ниже среднего
let totalEntries = 0;

for (let page of dv.pages(`"${CONFIG.NOTES_FOLDER}"`)) {
    let total = 0;
    let count = 0;

    for (const field of CONFIG.PROPERTY_FIELDS) {
        const value = parseFloat(page[field]);
        if (!isNaN(value)) {
            total += value;
            count++;
        }
    }

    if (count > 0) {
        totalEntries++;
        const average = total / count;
        if (average >= 7) {
            categoryData[0]++; // Отличное
        } else if (average >= 6) {
            categoryData[1]++; // Хорошее
        } else if (average >= 4) {
            categoryData[2]++; // Среднее
        } else {
            categoryData[3]++; // Ниже среднего
        }
    }
}

// Подготовка данных для графика
const labels = CONFIG.LABELS;
const data = categoryData;
const colors = CONFIG.COLORS;

// Добавляем подпись для диаграммы
const title = document.createElement("h3");
title.textContent = CONFIG.CHART_TITLE;
title.style.textAlign = "center";
title.style.marginBottom = "20px";
this.container.appendChild(title);

// Создание элемента canvas
const canvas = document.createElement("canvas");
canvas.width = CONFIG.CHART_DIMENSIONS.width;
canvas.height = CONFIG.CHART_DIMENSIONS.height;
this.container.appendChild(canvas);

const ctx = canvas.getContext("2d");

// Функция для построения круговой диаграммы
function drawPieChart(ctx, data, labels, colors, radius) {
    const total = data.reduce((sum, value) => sum + value, 0);
    let startAngle = -0.5 * Math.PI; // Начинаем с верхней точки

    data.forEach((value, index) => {
        if (value === 0) return; // Пропускаем категории с нулевым значением

        const sliceAngle = (value / total) * 2 * Math.PI;

        ctx.beginPath();
        ctx.moveTo(canvas.width / 2, canvas.height / 2); // Центр круга
        ctx.arc(canvas.width / 2, canvas.height / 2, radius, startAngle, startAngle + sliceAngle);
        ctx.closePath();
        ctx.fillStyle = colors[index];
        ctx.fill();

        startAngle += sliceAngle;
    });

    // Добавляем подписи
    startAngle = -0.5 * Math.PI;
    data.forEach((value, index) => {
        if (value === 0) return;

        const sliceAngle = (value / total) * 2 * Math.PI;
        const textAngle = startAngle + sliceAngle / 2;
        const textX = canvas.width / 2 + Math.cos(textAngle) * (radius * 0.7);
        const textY = canvas.height / 2 + Math.sin(textAngle) * (radius * 0.7);

        ctx.fillStyle = "black";
        ctx.font = "14px Arial";
        ctx.textAlign = "center";
        ctx.textBaseline = "middle";
        ctx.fillText(labels[index], textX, textY);

        startAngle += sliceAngle;
    });
}

// Улучшаем внешний вид (добавляем окружность и тень)
function styleChart(ctx, radius) {
    ctx.beginPath();
    ctx.arc(canvas.width / 2, canvas.height / 2, radius, 0, 2 * Math.PI);
    ctx.lineWidth = 2;
    ctx.strokeStyle = "#333";
    ctx.stroke();

    ctx.shadowColor = "rgba(0, 0, 0, 0.3)";
    ctx.shadowBlur = 15;
    ctx.shadowOffsetX = 5;
    ctx.shadowOffsetY = 5;
}

// Рисуем диаграмму
styleChart(ctx, CONFIG.CHART_RADIUS);
drawPieChart(ctx, data, labels, colors, CONFIG.CHART_RADIUS);

```

```dataviewjs
// Конфигурация
const PROPERTY_FIELDS = ["sport", "Mood", "cognitive", "sleep"]; // Поля для среднего
const NOTES_FOLDER = "Daily"; // Папка с заметками

// Сбор данных из заметок
const monthlyData = Array(12).fill(0).map(() => ({ total: 0, count: 0 }));

for (let page of dv.pages(`"${NOTES_FOLDER}"`)) {
    const dateMatch = page.file.name.match(/(\d{4})-(\d{2})-\d{2}/);
    if (!dateMatch) continue;

    const month = parseInt(dateMatch[2], 10) - 1; // Месяц
    let total = 0, count = 0;

    for (const field of PROPERTY_FIELDS) {
        const value = parseFloat(page[field]);
        if (!isNaN(value)) {
            total += value;
            count++;
        }
    }

    if (count > 0) {
        monthlyData[month].total += total / count;
        monthlyData[month].count++;
    }
}

// Расчёт среднего по месяцам
const averages = monthlyData.map(data => data.count > 0 ? data.total / data.count : 0);

// Рендер графика
const canvas = document.createElement("canvas");
canvas.width = 800;
canvas.height = 400;
this.container.appendChild(canvas);

const ctx = canvas.getContext("2d");

// Настройки графика
const labels = ["Янв", "Фев", "Мар", "Апр", "Май", "Июн", "Июл", "Авг", "Сен", "Окт", "Ноя", "Дек"];
const padding = 60;
const chartWidth = canvas.width - padding * 2;
const chartHeight = canvas.height - padding * 2;
const maxDataValue = Math.max(...averages);
const yAxisInterval = Math.ceil(maxDataValue / 5);
const gridColor = "#ddd";
const textColor = "#333";
const graphLineColor = "rgba(75, 192, 192, 1)";
const graphPointColor = "rgba(75, 192, 192, 1)";
const backgroundColor = "#f9f9f9";

// Рисуем фон
ctx.fillStyle = backgroundColor;
ctx.fillRect(0, 0, canvas.width, canvas.height);

// Сетка
ctx.strokeStyle = gridColor;
ctx.lineWidth = 1;

// Горизонтальные линии сетки
for (let i = 0; i <= 5; i++) {
    const y = canvas.height - padding - (chartHeight * (i / 5));
    ctx.beginPath();
    ctx.moveTo(padding, y);
    ctx.lineTo(canvas.width - padding, y);
    ctx.stroke();
}

// Вертикальные линии сетки
for (let i = 0; i < labels.length; i++) {
    const x = padding + (chartWidth / (labels.length - 1)) * i;
    ctx.beginPath();
    ctx.moveTo(x, padding);
    ctx.lineTo(x, canvas.height - padding);
    ctx.stroke();
}

// Оси
ctx.strokeStyle = textColor;
ctx.lineWidth = 2;

// Ось Y
ctx.beginPath();
ctx.moveTo(padding, padding);
ctx.lineTo(padding, canvas.height - padding);
ctx.stroke();

// Ось X
ctx.beginPath();
ctx.moveTo(padding, canvas.height - padding);
ctx.lineTo(canvas.width - padding, canvas.height - padding);
ctx.stroke();

// Подписи осей
ctx.font = "14px Arial";
ctx.fillStyle = textColor;
ctx.textAlign = "center";

// Подписи для Y
ctx.textAlign = "right";
ctx.textBaseline = "middle";
for (let i = 0; i <= 5; i++) {
    const yValue = yAxisInterval * i;
    const y = canvas.height - padding - (chartHeight * (yValue / maxDataValue));
    ctx.fillText(yValue.toFixed(1), padding - 10, y);
}

// Подписи для X
ctx.textAlign = "center";
ctx.textBaseline = "top";
labels.forEach((label, index) => {
    const x = padding + (chartWidth / (labels.length - 1)) * index;
    ctx.fillText(label, x, canvas.height - padding + 10);
});

// Линия графика
ctx.beginPath();
ctx.strokeStyle = graphLineColor;
ctx.lineWidth = 3;

averages.forEach((value, index) => {
    const x = padding + (chartWidth / (labels.length - 1)) * index;
    const y = canvas.height - padding - (chartHeight * (value / maxDataValue));
    if (index === 0) {
        ctx.moveTo(x, y);
    } else {
        ctx.lineTo(x, y);
    }
});

ctx.stroke();

// Точки на графике с подписями
averages.forEach((value, index) => {
    const x = padding + (chartWidth / (labels.length - 1)) * index;
    const y = canvas.height - padding - (chartHeight * (value / maxDataValue));

    // Рисуем точку
    ctx.beginPath();
    ctx.arc(x, y, 6, 0, 2 * Math.PI);
    ctx.fillStyle = graphPointColor;
    ctx.fill();
    ctx.strokeStyle = "#fff";
    ctx.lineWidth = 2;
    ctx.stroke();

    // Подписываем точку
    ctx.font = "12px Arial";
    ctx.fillStyle = textColor;
    ctx.textAlign = "center";
    ctx.textBaseline = "bottom";
    ctx.fillText(value.toFixed(2), x, y - 8);
});

// Заголовок
ctx.font = "18px Arial";
ctx.fillStyle = textColor;
ctx.textAlign = "center";
ctx.fillText("Среднее качество дня за год", canvas.width / 2, padding / 2);

```