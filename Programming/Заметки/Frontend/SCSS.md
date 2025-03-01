## Что такое SCSS?

SCSS (Sassy CSS) — это препроцессор CSS, который добавляет переменные, вложенность, миксины и другие удобные функции для упрощения и структурирования стилей. SCSS позволяет писать более чистый и удобочитаемый код.

### Преимущества SCSS:

- Использование переменных (`$color-primary: #3498db;`)
    
- Вложенные селекторы
    
- Миксины (`@mixin button-style {}`)
    
- Импорт файлов (`@import 'variables';`)
    
- Наследование (`@extend .btn;`)
    

## Установка SCSS

Если ваш проект использует npm, установите `sass`:

```
npm install -g sass
```

Компиляция SCSS в CSS:

```
sass styles.scss styles.css
```    

### Пример SCSS-кода с [[BEM methodology]]:

```
$primary-color: #3498db;

.button {
    background-color: $primary-color;
    color: white;
    padding: 10px 20px;
    border: none;
    cursor: pointer;
    
    &__icon {
        margin-right: 5px;
    }
    
    &--primary {
        background-color: darken($primary-color, 10%);
    }
    
    &--disabled {
        background-color: gray;
        cursor: not-allowed;
    }
}
```

### Итог

Использование SCSS в сочетании с методологией BEM делает код более понятным, гибким и масштабируемым. SCSS упрощает написание стилей, а BEM помогает организовать их структуру.