Вот готовый README.md в чистом Markdown-формате:

```markdown
# LifeTracker - Система персональной аналитики в Obsidian

![Obsidian Version](https://img.shields.io/badge/Obsidian-1.4%2B-blueviolet)
![Dataview Plugin](https://img.shields.io/badge/Plugin-Dataview-4B32C3)

Система для трекинга персональных метрик с визуализацией данных через Obsidian и плагин Dataview.

## Требования
- [Obsidian](https://obsidian.md) v1.4+
- Плагины:
  - [Templater](https://github.com/SilentVoid13/Templater) v1.21.0+
  - [Dataview](https://github.com/blacksmithgu/obsidian-dataview) v0.5.45+

## Установка

### 1. Инициализация хранилища
```bash
mkdir MyLifeTracker
cd MyLifeTracker
mkdir Daily
```

### 2. Настройка Templater
1. Установите плагин Templater через Community Plugins
2. Создайте шаблон `Templates/DailyNote.md`:
```markdown
---
created: {{date:YYYY-MM-DD}}
---

# {{date:YYYY-MM-DD}}
Sleep: 
Mood: 
Sport: 
Cognitive: 
```

3. Конфигурация Templater (`.obsidian/templater.json`):
```json
{
  "template_folder": "Templates",
  "auto_jump_to_cursor": true,
  "date_format": "YYYY-MM-DD"
}
```

## Структура данных
Ежедневные заметки должны содержать метрики в формате:
```markdown
Sleep: 7  # 1-10
Mood: 8   # 1-10 
Sport: 3   # 1-10
Cognitive: 6  # 1-10
```

## Визуализация данных

### Calendar.md
```dataviewjs
const HUE_RED = 0;
const HUE_GREEN = 120;
const pages = dv.pages('"Daily"').where(p => p.Sleep);

dv.span("## Sleep Calendar\n");
dv.span("🔴 Low | 🟢 High\n\n");

for(let month = 0; month < 12; month++) {
  const monthDays = pages.where(p => p.file.day.month === month);
  const heatmap = [...Array(31)].map((_, i) => {
    const day = monthDays.find(p => p.file.day.day === i+1);
    return day ? colorBlock(day.Sleep) : "⬛";
  });
  dv.span(`**${month+1}:** ${heatmap.join("")}\n\n`);
}

function colorBlock(value) {
  const hue = lerp(HUE_RED, HUE_GREEN, value/10);
  return `<span style="color:hsl(${hue},100%,50%)">■</span>`;
}
```

### PieChart.md
```dataviewjs
const data = {
  "Excellent": dv.pages('"Daily"').filter(p => p.Mood >= 8).length,
  "Good": dv.pages('"Daily"').filter(p => p.Mood >=5 && p.Mood <8).length,
  "Poor": dv.pages('"Daily"').filter(p => p.Mood <5).length
};

new Chart(
  dv.current().container,
  {
    type: 'pie',
    data: {
      labels: Object.keys(data),
      datasets: [{data: Object.values(data)}]
    }
  }
);
```

### YearGraph.md
```dataviewjs
const months = [...Array(12)].map((_,i) => 
  dv.pages('"Daily"')
    .filter(p => p.file.day.year == dv.current().file.day.year)
    .filter(p => p.file.day.month == i)
    .map(p => p.Sleep)
);

dv.span("## Monthly Sleep Trends\n");
dv.plot(
  months.map(m => m.length ? m.reduce((a,b)=>a+b)/m.length : 0),
  {height: 300, width: 800}
);
```

## Кастомизация
1. **Изменение метрик**:
```dataviewjs
dv.pages('"Daily"').filter(p => p.[YOUR_METRIC])
```

2. **Настройка цветов**:
```javascript
// Для календаря
const HUE_RED = 0;    // 0-360
const HUE_GREEN = 120;
```

## Структура проекта
```
MyLifeTracker/
├── Daily/
│   └── YYYY-MM-DD.md
├── Templates/
│   └── DailyNote.md
├── Calendar.md
├── PieChart.md
└── YearGraph.md
```
