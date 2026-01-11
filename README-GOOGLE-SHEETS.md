# Инструкция по настройке интеграции с Google Sheets

Эта инструкция поможет вам настроить отправку данных из анкеты гостя в Google Таблицу.

## Шаг 1: Создание Google Таблицы

1. Откройте [Google Sheets](https://sheets.google.com)
2. Создайте новую таблицу
3. Назовите таблицу (например, "Анкеты гостей - Свадьба Эмиль и Екатерина")
4. В первой строке создайте заголовки колонок:
   - **A1**: ФИО
   - **B1**: Присутствие
   - **C1**: Алкоголь
   - **D1**: Остаться на ночь
   - **E1**: Дата/Время отправки

## Шаг 2: Создание Google Apps Script

1. В вашей Google Таблице нажмите **Расширения** (Extensions) → **Apps Script**
2. Откроется редактор кода Apps Script
3. Удалите весь существующий код и вставьте следующий:

```javascript
function doPost(e) {
  try {
    // Получаем данные из запроса
    const data = JSON.parse(e.postData.contents);
    
    // Получаем активную таблицу
    const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    
    // Добавляем новую строку с данными
    sheet.appendRow([
      data.fullName || '',
      data.attendance || '',
      data.alcohol || '',
      data.overnight || '',
      data.timestamp || new Date().toString()
    ]);
    
    // Возвращаем успешный ответ
    return ContentService.createTextOutput(
      JSON.stringify({
        'status': 'success',
        'message': 'Данные успешно сохранены'
      })
    ).setMimeType(ContentService.MimeType.JSON);
    
  } catch (error) {
    // В случае ошибки возвращаем сообщение об ошибке
    return ContentService.createTextOutput(
      JSON.stringify({
        'status': 'error',
        'message': error.toString()
      })
    ).setMimeType(ContentService.MimeType.JSON);
  }
}
```

4. Нажмите **Ctrl+S** (или **Cmd+S** на Mac) для сохранения
5. Назовите проект (например, "Wedding RSVP Handler")

## Шаг 3: Развертывание как веб-приложения

1. В редакторе Apps Script нажмите на кнопку **Развернуть** (Deploy) → **Новое развертывание** (New deployment)
2. Нажмите на значок настроек (⚙️) рядом с "Выбрать тип" и выберите **Веб-приложение** (Web app)
3. Заполните форму:
   - **Описание**: Свадьба RSVP Handler
   - **Выполнять от имени**: Меня (ваш email)
   - **У кого есть доступ**: Все (Anyone)
4. Нажмите **Развернуть** (Deploy)
5. В появившемся окне авторизуйтесь:
   - Нажмите **Авторизовать доступ**
   - Выберите ваш Google аккаунт
   - Нажмите **Дополнительно** → **Перейти к [имя проекта] (небезопасно)**
   - Нажмите **Разрешить**
6. После авторизации скопируйте **URL веб-приложения** (Web app URL)
   - Это будет URL вида: `https://script.google.com/macros/s/AKfycby.../exec`

## Шаг 4: Настройка HTML файла

1. Откройте файл `invitation-emil-katya.html` в текстовом редакторе
2. Найдите строку (примерно строка 872):
   ```javascript
   const GOOGLE_SCRIPT_URL = 'YOUR_GOOGLE_APPS_SCRIPT_URL';
   ```
3. Замените `'YOUR_GOOGLE_APPS_SCRIPT_URL'` на скопированный URL из шага 3:
   ```javascript
   const GOOGLE_SCRIPT_URL = 'https://script.google.com/macros/s/AKfycby.../exec';
   ```
4. Сохраните файл

## Шаг 5: Тестирование

1. Откройте `invitation-emil-katya.html` в браузере
2. Заполните форму анкеты гостя
3. Нажмите кнопку **ОТПРАВИТЬ**
4. Проверьте вашу Google Таблицу - должна появиться новая строка с данными

## Важные замечания

### Безопасность
- URL вашего веб-приложения будет публичным после развертывания
- Любой, у кого есть этот URL, сможет отправлять данные в вашу таблицу
- Рекомендуется периодически проверять данные в таблице на предмет спама

### Ограничения Google Apps Script
- Google Apps Script имеет квоты на выполнение:
  - 20 000 запросов в день
  - 6 минут времени выполнения на запрос
- Для свадебных приглашений этих лимитов более чем достаточно

### Если данные не сохраняются

1. **Проверьте URL**: Убедитесь, что URL скопирован полностью и без ошибок
2. **Проверьте права доступа**: В настройках развертывания должно быть "Все" (Anyone)
3. **Проверьте консоль браузера**: 
   - Откройте инструменты разработчика (F12)
   - Перейдите на вкладку Console
   - Попробуйте отправить форму и посмотрите на ошибки
4. **Проверьте выполнение скрипта**:
   - В редакторе Apps Script перейдите в раздел "Выполнения" (Executions)
   - Посмотрите логи выполнения последних запросов

### Альтернативный метод (если нужна более продвинутая обработка)

Если вам нужна более сложная логика (валидация, отправка email, etc.), можно расширить скрипт:

```javascript
function doPost(e) {
  try {
    const data = JSON.parse(e.postData.contents);
    const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    
    // Валидация данных
    if (!data.fullName || !data.attendance) {
      throw new Error('Не заполнены обязательные поля');
    }
    
    // Добавление данных
    sheet.appendRow([
      data.fullName,
      data.attendance,
      data.alcohol || 'Не указано',
      data.overnight,
      new Date().toLocaleString('ru-RU')
    ]);
    
    // Опционально: отправка email уведомления
    // MailApp.sendEmail({
    //   to: 'your-email@gmail.com',
    //   subject: 'Новая анкета гостя: ' + data.fullName,
    //   body: 'Присутствие: ' + data.attendance + '\nАлкоголь: ' + data.alcohol
    // });
    
    return ContentService.createTextOutput(
      JSON.stringify({
        'status': 'success',
        'message': 'Данные успешно сохранены'
      })
    ).setMimeType(ContentService.MimeType.JSON);
    
  } catch (error) {
    return ContentService.createTextOutput(
      JSON.stringify({
        'status': 'error',
        'message': error.toString()
      })
    ).setMimeType(ContentService.MimeType.JSON);
  }
}
```

## Поддержка

Если у вас возникли проблемы с настройкой, вы можете:
1. Проверить [документацию Google Apps Script](https://developers.google.com/apps-script)
2. Убедиться, что у вас есть права на создание и развертывание веб-приложений в Google Workspace






