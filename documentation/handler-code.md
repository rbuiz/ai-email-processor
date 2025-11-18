```javascript

const inputData = $input.first().json.textHtml;


// ДИАГНОСТИКА: что приходит на самом деле
console.log('=== RAW INPUT ===');
console.log('Type:', typeof inputData);
console.log('Keys:', inputData ? Object.keys(inputData) : 'No input');
console.log('Input data:', JSON.stringify(inputData, null, 2));

// Получаем HTML из разных возможных источников
let rawHtml = '';

if (typeof inputData === 'string') {
    rawHtml = inputData;
} else if (inputData && inputData.json) {
    // Пробуем все возможные поля
    rawHtml = inputData.json.html || 
              inputData.json.content || 
              inputData.json.text || 
              inputData.json.body ||
              inputData.json.data ||
              JSON.stringify(inputData.json);
}

console.log('Raw HTML length:', rawHtml.length);
console.log('First 500 chars:', rawHtml.substring(0, 500));

// ПРОСТОЙ И ЭФФЕКТИВНЫЙ МЕТОД
function fixEncoding(text) {
    if (!text) return '';
    
    // Метод 1: Пробуем разные кодировки
    let fixed = text;
    
    try {
        // Latin1 -> UTF-8 (самый частый случай)
        const latin1Fixed = Buffer.from(text, 'latin1').toString('utf8');
        if (latin1Fixed.includes('Битва') || latin1Fixed.match(/[А-Яа-я]/)) {
            fixed = latin1Fixed;
            console.log('Used Latin1 fix');
        }
    } catch(e) {}
    
    try {
        // Binary -> UTF-8  
        const binaryFixed = Buffer.from(text, 'binary').toString('utf8');
        if (binaryFixed.includes('Битва') || binaryFixed.match(/[А-Яа-я]/)) {
            fixed = binaryFixed;
            console.log('Used Binary fix');
        }
    } catch(e) {}
    
    // Метод 2: Ручные замены для частых случаев
    const replacements = [
        ['?', 'Д'], ['?', 'С'], ['?', ''], ['?', ''],
        ['?°', 'а'], ['?±', 'б'], ['??', 'в'], ['??', 'г'],
        ['??', 'д'], ['?µ', 'е'], ['?', 'с'], ['?·', 'з'],
        ['??', 'и'], ['??', 'й'], ['??', 'к'], ['?»', 'л'],
        ['??', 'м'], ['??', 'н'], ['??', 'о'], ['??', 'п'],
        ['?€', 'р'], ['??', 'с'], ['?‚', 'т'], ['??', 'у'],
        ['?„', 'ф'], ['?…', 'х'], ['?†', 'ц'], ['?‡', 'ч'],
        ['??', 'ш'], ['?‰', 'щ'], ['?‹', 'ы'], ['??', 'ь'],
        ['??', 'э'], ['??', 'ю'], ['??', 'я']
    ];
    
    replacements.forEach(([from, to]) => {
        fixed = fixed.split(from).join(to);
    });
    
    return fixed;
}

// ОСНОВНАЯ ОБРАБОТКА
let resultText = '';

if (rawHtml) {
    // 1. Исправляем кодировку ДО очистки HTML
    const fixedEncoding = fixEncoding(rawHtml);
    console.log('After encoding fix:', fixedEncoding.substring(0, 200));
    
    // 2. Очищаем HTML
    resultText = fixedEncoding
        .replace(/<script\b[^>]*>[\s\S]*?<\/script>/gi, '')
        .replace(/<style\b[^>]*>[\s\S]*?<\/style>/gi, '')
        .replace(/<[^>]*>/g, ' ')
        .replace(/\s+/g, ' ')
        .trim()

    resultText = resultText
      .replace(/[\n\r\t]/g, ' ')    // Убираем все переносы и табы
      .replace(/\s{2,}/g, ' ')      // Заменяем 2+ пробелов на один
      .replace(/[\s\u2000-\u200F\u2028-\u202F\u205F-\u206F\u3000\u3164\uFEFF]+/g, ' ')
      .trim();                      // Убираем пробелы по краям

  resultText = resultText
      .replace(/[^\P{C}\n]/gu, ' ')   // Убирает ВСЕ управляющие символы кроме переносов
      .split(/\r?\n/)                 // Разбивает по строкам
      .map(line => line.trim())       // Обрезает каждую строку
      .filter(line => line.length > 0 && !/^[\s\u200B]*$/.test(line)) // Убирает пустые
      .join(' ')                      // Собирает в одну строку
      .replace(/\s+/g, ' ')           // Убирает лишние пробелы
      .trim();
  
    console.log('After HTML cleanup:', resultText.substring(0, 200));
} else {
    resultText = 'No HTML data found';
}

// ВОЗВРАЩАЕМ РЕЗУЛЬТАТ
return [{
    json: {
        cleanText: resultText,
        originalLength: rawHtml.length,
        cleanLength: resultText.length,
        containsRussian: /[А-Яа-я]/.test(resultText),
        diagnostic: 'Check console for details'
    }
}];

```
