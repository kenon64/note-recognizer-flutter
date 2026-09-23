# ARCHITECTURE.md

## 1. Поток данных

```text
Микрофон
  ↓
Record stream (PCM16, 16kHz, mono)
  ↓
AudioCaptureService
  ↓
RMS + High-pass + Windowing
  ↓
FeatureExtractor
  ├── Dominant frequency estimate
  ├── Spectral profile (naive DFT-based)
  └── MFCC-like compact vector
  ↓
TemplateMatcher
  ├── Cosine Similarity
  └── Confidence threshold
  ↓
UI: note, keyboard highlight, chord aggregation
```

## 2. Распознавание

### 2.1 Silence gate

Сначала вычисляется RMS. Если уровень ниже порога, приложение считает, что звук отсутствует.

### 2.2 Pre-processing

- High-pass фильтр для подавления гула ниже 80 Гц
- Нормализация громкости для единообразия амплитуды
- усечение буфера до целевого размера

### 2.3 Features

Вектор признаков содержит:

- RMS
- dominant frequency estimate
- спектральный профиль по ключевым частотам
- нормализованный набор значений мощности по диапазону

### 2.4 Template Matching

Сравнение выполняется через cosine similarity между текущим вектором и вектором шаблона. Если значение выше порога, распознаётся конкретная нота.

Если similarity низкая, выполняется fallback-поиск по частоте (YIN-like estimation).

## 3. Аккорды

Аккорд собирается по FIFO-очереди последних нот. Если пауза между нотами превышает таймаут, очередь сбрасывается. Когда записано минимум 3 уникальные ноты, выполняется поиск по словарю аккордов.

## 4. Хранение

Шаблоны сохраняются в директории приложения:

```text
app_documents/templates/templates.json
```

Аудиофайлы сохраняются по пути вида:

```text
app_documents/templates/<note>.wav
```

## 5. Пороговые значения по умолчанию

- RMS threshold: 0.015
- Similarity threshold: 0.78
- Chord timeout: 1500 ms
- Noise warning: 0.12

## 6. Ограничения текущей реализации

- Для Android APK это локальный, легковесный прототип
- В реальном продакшене желательно добавить дополнительные DSP-алгоритмы и более крупную базу шаблонов
- Аудиодрайверы Android могут вносить задержку, поэтому реальный latency лучше смотреть на устройстве
