(В архиве содержатся файлы агентов, доски и модели для олламы и пипер/The archive contains agent files, boards and models for the ollama and piper)

# -Simba-Voice-Assistant-Distributed-System-with-Blackboard-Architecture
это пример - рабочая базовая модель, простая и универсальная, разбирайте ее или допиливайте | This is a working base model, simple and universal - disassemble or refine it as needed
Это мой первый проэкт, мне помогли вдохновиться, научиться и это сделать: хороший человек Паша и ИИ: GitHub Copilot, Qwen, DeepSeek - за что я выражаю большую благодарность!
Выложил скорее чтоб опубликовать черную доску, так как думаю это простое рабочее решение. (которое можно при желании дообграйдить для мега многозадачных общений любого количества агентов - имхо, не проверял пока) 

This is my first project, and I'm deeply grateful to a great person named Pasha, and to AI tools—GitHub Copilot, Qwen, and DeepSeek—for inspiring me, teaching me, and helping me bring it to life.  
I'm sharing it mainly to publish the "blackboard" concept, as I believe it's a simple and practical solution that can be further upgraded for highly complex, multi-agent conversations—if desired (just my opinion, haven't tested it yet).

activate:
WAKE_WORDS = [
    "симб", "симба", "симби", "симка", "симпа", "сим", 
    "симбик", "симон", "симоне", "симоней", "симонейка",
    "simba", "simbi", "simby", "simbya", "simbee",
    "симбоче", "симбочка", "симбочик", "симбочок"
]

def is_wake_word(text):
    """Проверяет, содержит ли текст слово-активатор"""
    text = " " + text.lower() + " "  # Добавляем пробелы для проверки границ слов
    return any(f" {word} " in text for word in WAKE_WORDS)

# Голосовой ассистент "Симба": распределенная система с архитектурой "черной доски"

## Общее описание

"Симба" — это модульный голосовой ассистент с распределенной архитектурой, построенный на принципе "черной доски". Система состоит из независимых компонентов, которые взаимодействуют через общую файловую базу данных (blackboard.jsonl), публикуя и обрабатывая сообщения.

### Основные функции
- Непрерывное прослушивание микрофона с активацией по ключевому слову "симба"
- Поддержка команды остановки ("Симба, прервемся")
- Распределенная обработка: распознавание речи, генерация ответов, озвучка
- Автоматический выбор рабочего микрофона
- Интеграция с локальной LLM через Ollama
- Синтез речи через Piper TTS

### Принцип работы
1. Коммуникатор обнаруживает активацию по ключевому слову и отправляет текст на обработку
2. Ollama-агент генерирует текстовый ответ
3. Piper-агент преобразует текст в речь и воспроизводит его
4. Все компоненты взаимодействуют через файловую черную доску

## Архитектура системы

### 1. Черная доска (Blackboard)

Центральный механизм межпроцессного взаимодействия, реализованный как файл журнала сообщений (blackboard.jsonl).

**Ключевые особенности:**
- Каждое сообщение имеет уникальный ID, временную метку, автора, топик, теги и содержание
- Реализует шаблон "публикация-подписка" через метод monitor()
- Содержит поле processed_by для отслеживания обработанных сообщений
- Обеспечивает согласованность данных через файловые блокировки (fcntl)
- Поддерживает автоматическую очистку старых сообщений

### 2. Коммуникатор (Communicator)

**Основная функция:** Распознавание речи и активация системы.

**Технические детали:**
- Непрерывное прослушивание микрофона с автоматическим выбором устройства
- Распознавание речи через Google Speech API
- Фильтрация собственных ответов (предотвращение реакции на собственную озвучку)
- Публикация распознанного текста на черной доске
- Поддержка команды остановки

**Технологии:** speech_recognition, asyncio, pyaudio

### 3. Ollama-агент

**Основная функция:** Генерация ответов с использованием локальной языковой модели.

**Технические детали:**
- Подписка на команды от Коммуникатора
- Взаимодействие с API Ollama через HTTP-запросы
- Генерация текстовых ответов с учетом контекста
- Публикация ответов на черной доске
- Управление таймаутами (рекомендуется 300 секунд для сложных запросов)

**Технологии:** requests, Ollama API

### 4. Piper-агент

**Основная функция:** Синтез речи из текстовых ответов.

**Технические детали:**
- Подписка на текстовые ответы от Ollama-агента
- Использование Piper для преобразования текста в речь
- Поддержка русскоязычных моделей (например, ru_RU-irina-medium)
- Воспроизведение аудио через системный аудиоплеер
- Управление временем охлаждения на основе длины ответа

**Технологии:** subprocess, Piper TTS

### 5. Агентская регистрация (Agent Registry)

**Основная функция:** Управление жизненным циклом агентов.

**Технические детали:**
- Регистрация агентов с указанием их фильтров подписки
- Отправка "сердцебиений" для мониторинга активности
- Определение доступных агентов для обработки сообщений

## Установка и настройка

### Требования к системе
- ОС: Ubuntu 22.04 или новее
- Память: минимум 6 ГБ (рекомендуется 16 ГБ)
- Дисковое пространство: 10+ ГБ
- Микрофон и динамики

### Установка зависимостей

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y git python3 python3-pip python3-venv \
    build-essential libasound2-dev portaudio19-dev libportaudio2 \
    ffmpeg libsndfile1 swig
```

### Установка Python-окружения

```bash
mkdir -p ~/simba/project/agents
cd ~/simba
python3.10 -m venv venv
source venv/bin/activate
pip install --upgrade pip
pip install speechrecognition pyaudio requests webrtcvad psutil
```

### Установка Ollama

```bash
curl -L https://ollama.ai/install.sh | sh
sudo systemctl enable ollama
sudo systemctl start ollama
ollama pull llama3
```

### Установка Piper TTS

```bash
cd ~/simba
git clone https://github.com/rhasspy/piper.git
cd piper
make build

# Установка русской модели
cd ~/simba
mkdir -p piper_models/ru_RU-irina-medium
cd piper_models/ru_RU-irina-medium
wget https://github.com/rhasspy/piper/releases/download/v1.0.0/ru_RU-irina-medium.onnx
wget https://github.com/rhasspy/piper/releases/download/v1.0.0/ru_RU-irina-medium.onnx.json
```

### Настройка проекта

Создайте структуру проекта:

```
project/
├── agents/
│   ├── communicator.py
│   ├── ollama_agent.py
│   └── piper_agent.py
├── core/
│   ├── blackboard.py
│   └── agent_registry.py
```

Убедитесь, что пути к Piper и моделям в piper_agent.py корректны:

```python
PIPER_BINARY = "/home/ваше_имя/simba/piper/piper"
PIPER_MODEL = "/home/ваше_имя/simba/piper_models/ru_RU-irina-medium/ru_RU-irina-medium.onnx"
```

### Запуск системы

Запустите компоненты в отдельных терминалах:

```bash
# Терминал 1: Активация окружения
cd ~/simba
source venv/bin/activate

# Терминал 1: Ollama (если не запущен автоматически)
(выключаю вот так: systemctl --user stop ollama)
включаю:
OLLAMA_MODELS=/simba/ollama_models ollama serve > /dev/null 2>&1 &

# Терминал 2: Ollama-агент
cd ~/simba
source venv/bin/activate
cd /project
python agents/ollama_agent.py

# Терминал 3: Piper-агент
cd ~/simba
source venv/bin/activate
cd /project
python agents/piper_agent.py

# Терминал 4: Коммуникатор
cd ~/simba
source venv/bin/activate
cd /project
python agents/communicator.py
```


## Распространенные проблемы и решения

### 1. Проблемы с микрофоном
- Проверьте системные настройки звука
- В communicator.py можно указать конкретный микрофон: `python agents/communicator.py --mic 2`

### 2. Таймауты Ollama
Если сложные запросы не обрабатываются, увеличьте таймаут в ollama_agent.py:
```python
response = requests.post(OLLAMA_URL, json={...}, timeout=300)  # Было 60
```

### 3. Проблемы с Python 3.12
- Используйте Python 3.10 как основной интерпретатор
- Или отключите WebRTC в communicator.py

### 4. Установка Piper
Обратите внимание, что оригинальный репозиторий Piper (rhasspy/piper) был перемещен в OHF-Voice/piper1-gpl и заархивирован 6 октября 2025 года. Для новых установок рекомендуется использовать актуальный репозиторий.

## Расширение функциональности

### 1. Добавление новых агентов
- Создайте новый модуль, наследующий базовый класс агента
- Определите фильтры подписки на нужные топики
- Реализуйте обработку сообщений

### 2. Замена компонентов
- Ollama можно заменить на другой LLM-сервис
- Piper TTS можно заменить на любой другой TTS-движок
- Распознавание речи можно реализовать через другие API или локальные модели

### 3. Масштабирование
- Распределите агенты по разным машинам
- Реализуйте синхронизацию между несколькими экземплярами черной доски
- Добавьте механизмы отказоустойчивости

## Заключение

"Симба" представляет собой готовую к использованию систему голосового ассистента с модульной архитектурой. Основные преимущества проекта:
- Распределенная обработка через шину сообщений "черная доска"
- Локальная работа без зависимости от облачных сервисов
- Гибкость и возможность замены отдельных компонентов
- Поддержка русского языка на всех этапах обработки

Проект может быть использован как основа для создания голосовых интерфейсов, интеграции с умным домом или как учебный пример распределенных систем с использованием современных LLM.

# Simba Voice Assistant: Distributed System with Blackboard Architecture

## Overview

Simba is a modular voice assistant with a distributed architecture built on the "blackboard" principle. The system consists of independent components that interact through a shared file-based message board (blackboard.jsonl), publishing and processing messages.

### Key Features
- Continuous microphone listening with keyword activation ("simba")
- Stop command support ("simba, pause")
- Distributed processing: speech recognition, response generation, speech synthesis
- Automatic microphone selection
- Integration with local LLM through Ollama
- Text-to-speech synthesis using Piper TTS

### How It Works
1. Communicator detects activation by keyword and sends text for processing
2. Ollama agent generates text responses
3. Piper agent converts text to speech and plays it
4. All components interact through the file-based blackboard

## System Architecture

### 1. Blackboard

The central inter-process communication mechanism implemented as a message log file (blackboard.jsonl).

**Key characteristics:**
- Each message has a unique ID, timestamp, author, topic, tags, and content
- Implements "publish-subscribe" pattern through the monitor() method
- Contains processed_by field to track processed messages
- Ensures data consistency through file locks (fcntl)
- Supports automatic cleanup of old messages

### 2. Communicator

**Primary function:** Speech recognition and system activation.

**Technical details:**
- Continuous microphone listening with automatic device selection
- Speech recognition using Google Speech API
- Filtering of self-responses (preventing reaction to own output)
- Publishing recognized text to the blackboard
- Stop command support

**Technologies used:** speech_recognition, asyncio, pyaudio

### 3. Ollama Agent

**Primary function:** Response generation using local language models.

**Technical details:**
- Subscribes to commands from the Communicator
- Interacts with Ollama API through HTTP requests
- Generates text responses considering context
- Publishes responses to the blackboard
- Timeout management (recommended 300 seconds for complex queries)

**Technologies used:** requests, Ollama API

### 4. Piper Agent

**Primary function:** Text-to-speech synthesis from textual responses.

**Technical details:**
- Subscribes to text responses from Ollama agent
- Uses Piper for text-to-speech conversion
- Supports Russian language models (e.g., ru_RU-irina-medium)
- Audio playback through system audio player
- Cooldown management based on response length

**Technologies used:** subprocess, Piper TTS

> **Note:** The original Piper repository (rhasspy/piper) has been moved to [OHF-Voice/piper1-gpl](https://github.com/OHF-Voice/piper1-gpl) and was archived by the owner on October 6, 2025. For new installations, please use the current repository.

### 5. Agent Registry

**Primary function:** Managing the agent lifecycle.

**Technical details:**
- Registers agents with their subscription filters
- Sends "heartbeat" signals for activity monitoring
- Determines available agents for message processing

## Installation and Setup

### System Requirements
- OS: Ubuntu 22.04 or newer
- Memory: Minimum 6 GB (16 GB recommended)
- Disk space: 10+ GB
- Microphone and speakers

### Dependency Installation

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y git python3 python3-pip python3-venv \
    build-essential libasound2-dev portaudio19-dev libportaudio2 \
    ffmpeg libsndfile1 swig
```

### Python Environment Setup

```bash
mkdir -p ~/simba/project/agents
cd ~/simba
python3.10 -m venv venv
source venv/bin/activate
pip install --upgrade pip
pip install speechrecognition pyaudio requests webrtcvad psutil
```

### Ollama Installation

```bash
curl -L https://ollama.ai/install.sh | sh
sudo systemctl enable ollama
sudo systemctl start ollama
ollama pull llama3
```

### Piper TTS Installation

```bash
cd ~/simba
git clone https://github.com/OHF-Voice/piper1-gpl.git piper
cd piper
make build

# Install Russian language model
cd ~/simba
mkdir -p piper_models/ru_RU-irina-medium
cd piper_models/ru_RU-irina-medium
wget https://github.com/rhasspy/piper/releases/download/v1.0.0/ru_RU-irina-medium.onnx
wget https://github.com/rhasspy/piper/releases/download/v1.0.0/ru_RU-irina-medium.onnx.json
```

### Project Configuration

Create the project structure:

```
project/
├── agents/
│   ├── communicator.py
│   ├── ollama_agent.py
│   └── piper_agent.py
├── core/
│   ├── blackboard.py
│   └── agent_registry.py
```

Ensure paths to Piper and models in piper_agent.py are correct:

```python
PIPER_BINARY = "/home/your_username/simba/piper/piper"
PIPER_MODEL = "/home/your_username/simba/piper_models/ru_RU-irina-medium/ru_RU-irina-medium.onnx"
```

### Running the System

Launch components in separate terminals:

```bash
# Терминал 1: Ollama (если не запущен автоматически)
(выключаю вот так: systemctl --user stop ollama)
включаю:
OLLAMA_MODELS=/simba/ollama_models ollama serve > /dev/null 2>&1 &

# Терминал 2: Ollama-агент
cd ~/simba
source venv/bin/activate
cd /project
python agents/ollama_agent.py

# Терминал 3: Piper-агент
cd ~/simba
source venv/bin/activate
cd /project
python agents/piper_agent.py

# Терминал 4: Коммуникатор
cd ~/simba
source venv/bin/activate
cd /project
python agents/communicator.py
```

## Common Issues and Solutions

### 1. Microphone Problems
- Check system sound settings
- In communicator.py, specify a particular microphone: `python agents/communicator.py --mic 2`

### 2. Ollama Timeouts
If complex queries aren't processed, increase the timeout in ollama_agent.py:
```python
response = requests.post(OLLAMA_URL, json={...}, timeout=300)  # Was 60
```

### 3. Python 3.12 Issues
- Use Python 3.10 as the primary interpreter
- Or disable WebRTC in communicator.py

### 4. Piper Installation
Note that the original Piper repository (rhasspy/piper) was moved to OHF-Voice/piper1-gpl and archived on October 6, 2025. For new installations, use the current repository as shown in the installation instructions.

## Extending Functionality

### 1. Adding New Agents
- Create a new module inheriting from the base agent class
- Define subscription filters for relevant topics
- Implement message processing

### 2. Component Replacement
- Ollama can be replaced with another LLM service
- Piper TTS can be replaced with any other TTS engine
- Speech recognition can be implemented through other APIs or local models

### 3. Scaling
- Distribute agents across different machines
- Implement synchronization between multiple blackboard instances
- Add fault tolerance mechanisms

## Conclusion

Simba provides a ready-to-use voice assistant system with modular architecture. Key advantages of the project:
- Distributed processing through the "blackboard" message bus
- Local operation without dependency on cloud services
- Flexibility and ability to replace individual components
- Full Russian language support at all processing stages

The project can serve as a foundation for voice interfaces, smart home integration, or as an educational example of distributed systems using modern LLMs.
