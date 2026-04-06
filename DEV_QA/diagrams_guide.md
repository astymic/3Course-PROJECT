# 📘 Гайд: як працювати з діаграмами проєкту

## Де дивитися діаграми?

### Варіант 1: GitHub (найпростіший ✅)
Просто відкрий файл на GitHub — діаграми рендеряться **автоматично**:
- [architecture.md](https://github.com/astymic/3Course-PROJECT/blob/main/DEV_QA/architecture.md)
- [jira_workflow.md](https://github.com/astymic/3Course-PROJECT/blob/main/DEV_QA/jira_workflow.md)

GitHub підтримує Mermaid нативно — нічого встановлювати не потрібно.

---

### Варіант 2: VS Code (локально)
Встанови розширення, і діаграми будуть видні прямо в превʼю:
1. Відкрий VS Code
2. Встанови розширення: **"Markdown Preview Mermaid Support"**
   - `Ctrl+Shift+X` → пошук → `bierner.markdown-mermaid` → Install
3. Відкрий `.md` файл → натисни `Ctrl+Shift+V` (Markdown Preview)
4. Всі `mermaid`-блоки відобразяться як схеми

---

### Варіант 3: Mermaid Live Editor (онлайн)
Якщо потрібно відредагувати або експортувати як PNG/SVG:
1. Відкрий: **https://mermaid.live**
2. Скопіюй вміст блоку ` ```mermaid ... ``` ` (без зворотних лапок)
3. Вставь у лівий редактор — справа зʼявиться діаграма
4. Кнопки зверху: **PNG**, **SVG**, **Copy link**

---

## Які діаграми є в проєкті

| Файл | Тип діаграми | Що показує |
|---|---|---|
| `architecture.md` | `graph TB` | Трирівнева архітектура (клієнт → сервер → БД) |
| `architecture.md` | `graph LR` | Взаємодія компонентів (Frontend ↔ Backend ↔ API) |
| `architecture.md` | `erDiagram` | ER-схема бази даних (7 таблиць) |
| `architecture.md` | `sequenceDiagram` | Потік замовлення (покупець → оплата → доставка) |
| `jira_workflow.md` | `stateDiagram` | Workflow задач (To Do → Done) |
| `jira_workflow.md` | `mindmap` | Карта всіх 9 Epics та 28 Stories |
| `jira_workflow.md` | `gantt` | Розклад задач по 3 спринтах |

---

## Як експортувати для презентації

### PNG / SVG (для слайдів):
1. Відкрий https://mermaid.live
2. Вставь код діаграми
3. Натисни **PNG** або **SVG** → збережи
4. Встав у PowerPoint / Google Slides

### PDF (для документації):
1. Відкрий `.md` файл у VS Code з Mermaid-розширенням
2. Встанови розширення **"Markdown PDF"** (`yzane.markdown-pdf`)
3. `Ctrl+Shift+P` → "Markdown PDF: Export (pdf)"
4. Отримаєш PDF з усіма діаграмами

---

## Швидке редагування

Якщо потрібно змінити діаграму (наприклад додати нову Story):

1. Відкрий `.md` файл
2. Знайди потрібний блок ` ```mermaid `
3. Відредагуй текст всередині (Mermaid використовує простий текстовий синтаксис)
4. Збережи → перевір на GitHub або в превʼю VS Code

**Приклад:** додати нову Story в mindmap:
```
# Було:
        🛒 Epic 4: Кошик
            LIL-13 Додати в кошик

# Стало:
        🛒 Epic 4: Кошик
            LIL-13 Додати в кошик
            LIL-29 Змінити кількість у кошику    ← нова строка
```
