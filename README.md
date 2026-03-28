# Concurrent Cryptography Service

>High-performance concurrent service for encrypting sensitive data (e.g. bank card numbers) using **AES-GCM** with strong guarantees of correctness, security and scalability.

> **Note:** The implementation source code and project files are not publicly available due to NDA and course confidentiality policies.

---

## Overview

Проект представляет собой сервис для **массового шифрования банковских карт** при ротации ключей.

Основные требования:
- высокая производительность (обработка большого количества данных)
- безопасность (криптографически корректная реализация)
- потокобезопасность
- отсутствие лишних аллокаций и деградации GC

---

## Key Features

- **AES-GCM encryption (AEAD)**  
  Шифрование с аутентификацией данных (защита от подмены)

- **Concurrent processing (worker pool)**  
  Параллельная обработка карт с контролем числа воркеров

- **Zero-overhead optimizations**  
  Минимизация аллокаций и работа с памятью через `unsafe`

- **Functional Options pattern**  
  Гибкая конфигурация количества воркеров

- **Deterministic output ordering**  
  Результат сохраняет порядок входных данных

---

## Architecture

Обработка разбита на несколько этапов:

Input (cards)
↓
Work partitioning
↓
Worker pool (goroutines)
↓
Encryption (AES-GCM)
↓
Encoding (hex)
↓
Output (ordered result)

### Worker model

- входной массив разбивается на чанки
- каждый воркер обрабатывает свой диапазон
- используется `sync.WaitGroup`
- исключены гонки за счет строгого разделения диапазонов

---

## Cryptography

Используется:
- **AES (Advanced Encryption Standard)**
- режим **GCM (Galois/Counter Mode)**

Почему GCM:
- обеспечивает **конфиденциальность + целостность (AEAD)**
- устойчив к подмене данных
- эффективно реализован аппаратно

### Nonce generation

Для каждого шифрования:
- генерируется уникальный nonce
- используется `crypto/rand` (CSPRNG)
- соблюдается требование уникальности для ключа

---

## Performance Optimizations

В проекте применены реальные low-level оптимизации:

- переиспользование буферов внутри воркеров
- минимизация аллокаций
- батчинг результатов в один слайс
- использование `unsafe`:
  - `unsafe.String`
  - `unsafe.Slice`
- контроль размера worker pool
- учет особенностей `crypto/rand` на разных ОС

---

## Concurrency

Реализовано:

- worker pool с динамическим распределением нагрузки
- балансировка задач между воркерами
- отсутствие mutex (за счет изоляции данных)
- потокобезопасная запись результата
