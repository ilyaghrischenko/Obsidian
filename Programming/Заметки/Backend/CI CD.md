## **Что такое CI/CD**

  

**CI/CD** (Continuous Integration / Continuous Delivery или Continuous Deployment) — это набор практик, позволяющих автоматизировать процесс интеграции кода, тестирования и доставки приложения в продакшн.

- **CI (Continuous Integration)** — непрерывная интеграция. Каждый коммит автоматически проверяется: проект собирается, запускаются юнит-тесты, статический анализ кода и другие проверки.
    
- **CD (Continuous Delivery)** — непрерывная доставка. После успешного CI приложение готово к выкатыванию на стенд или в продакшн вручную (по кнопке).
    
- **CD (Continuous Deployment)** — непрерывный деплой. После успешного CI изменения автоматически выкатываются в продакшн.
    

  

## **Зачем нужно CI/CD**

- Минимизирует человеческие ошибки при деплое.
    
- Обеспечивает быстрый фидбэк о качестве кода.
    
- Позволяет команде чаще выпускать обновления.
    
- Повышает надёжность и скорость разработки.
    
- Упрощает масштабирование команды.
    

  

## **Основные этапы CI/CD pipeline**

1. **Сборка (Build)** — компиляция и упаковка приложения.
    
2. **Тестирование (Test)** — запуск юнит, интеграционных, e2e тестов.
    
3. **Анализ качества кода** — линтеры, статический анализ, проверка стиля.
    
4. **Деплой (Deploy)**:
    
    - на тестовый стенд (staging);
        
    - на продакшн.
        
    

  

## **Примеры использования**

  

### **GitHub Actions (YAML конфигурация)**

``` yaml
name: CI/CD Pipeline

on:
  push:
    branches: ["main"]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Setup .NET
        uses: actions/setup-dotnet@v3
        with:
          dotnet-version: '8.0.x'
      - name: Restore dependencies
        run: dotnet restore
      - name: Build
        run: dotnet build --no-restore
      - name: Test
        run: dotnet test --no-build --verbosity normal

  deploy:
    needs: build-and-test
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to server
        run: |
          echo "Deploying application..."
          # здесь команды для деплоя (scp, docker, kubectl и т.д.)
```

### **Azure DevOps Pipeline**

``` yaml
trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

steps:
- task: DotNetCoreCLI@2
  inputs:
    command: 'restore'
    projects: '**/*.csproj'

- task: DotNetCoreCLI@2
  inputs:
    command: 'build'
    projects: '**/*.csproj'

- task: DotNetCoreCLI@2
  inputs:
    command: 'test'
    projects: '**/*Tests.csproj'
```

## **Лучшие практики**

- Запускать CI при каждом коммите/PR.
    
- Разделять пайплайн на этапы (build → test → deploy).
    
- Использовать staging среду перед продакшн деплоем.
    
- Держать пайплайн быстрым (до 10 минут на CI).
    
- Внедрять автоматический rollback при ошибке.
    

  

## **Когда использовать**

  

CI/CD стоит внедрять, когда:

- проект разрабатывает более одного человека;
    
- релизы происходят регулярно;
    
- важна стабильность и скорость доставки;
    
- используется DevOps-подход с контейнерами, облаками, микросервисами.
    

---

**Итог:** CI/CD — это обязательная практика в современной разработке, которая автоматизирует тестирование и деплой, снижает риск ошибок и ускоряет выпуск продукта.