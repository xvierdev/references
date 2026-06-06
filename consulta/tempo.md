# ⏳ Guia de Manipulação de Tempo em Python

Lidar com datas, horários e fusos é uma tarefa comum e cheia de armadilhas. Este guia cobre o módulo padrão `datetime`, os utilitários do módulo `time`, fusos de horário (`zoneinfo`), parsing, formatação e melhores práticas.

---

## 📑 Sumário
1. [📦 Visão Geral dos Módulos](#-visão-geral-dos-módulos)
2. [📅 `date`, `time` e `datetime`](#-date-time-e-datetime)
3. [⏱️ `timedelta` e aritmética](#-timedelta-e-aritmética)
4. [📝 Formatação e parsing de strings](#-formatação-e-parsing-de-strings)
5. [🌍 Timezones, `zoneinfo` e UTC](#-timezones-zoneinfo-e-utc)
6. [🕒 Módulo `time` e carimbos de relógio](#-módulo-time-e-carimbos-de-relógio)
7. [📆 `calendar` e operações de calendário](#-calendar-e-operações-de-calendário)
8. [✅ Boas práticas para aplicações](#-boas-práticas-para-aplicações)
9. [📚 Bibliotecas externas úteis](#-bibliotecas-externas-úteis)

---

## 📦 Visão Geral dos Módulos

- `datetime`: classe de alto nível para data, hora, timestamp e timezone.
- `time`: funções de baixo nível para relógio, dormência, conversão local/global e informações de fuso.
- `zoneinfo`: timezone nativo a partir do Python 3.9.
- `calendar`: utilitários de calendário e cálculo de feriados simples.

---

## 📅 `date`, `time` e `datetime`

### `date`
```python
from datetime import date

hoje = date.today()
ontem = hoje.replace(day=hoje.day - 1)
nasceu = date(1995, 5, 20)
print(hoje.isoformat())  # 2023-10-27
print(hoje.weekday())   # 0 = segunda, 6 = domingo
print(hoje.isoweekday())  # 1 = segunda, 7 = domingo
print(hoje.isocalendar())  # (ano, semana, dia)
```

### `time`
```python
from datetime import time

horario = time(14, 30, 45, 123456)
print(horario.isoformat())  # 14:30:45.123456
```

### `datetime`
```python
from datetime import datetime

agora = datetime.now()
agora_utc = datetime.utcnow()
manutencao = datetime(2023, 12, 25, 2, 0, 0)
print(agora.year, agora.month, agora.day)
print(agora.date())  # extrai date
print(agora.time())  # extrai time
```

### Criação e conversão
```python
from datetime import datetime

iso = datetime.fromisoformat('2023-10-27T14:30:45')
print(iso)

com_ts = datetime.fromtimestamp(1698424245)
print(com_ts)
```

### Métodos úteis
- `replace()`: altera ano, mês, dia, hora e timezone
- `combine(date, time)`: cria `datetime` a partir de `date` e `time`
- `today()`, `now()`, `utcnow()`, `fromtimestamp()`, `utcfromtimestamp()`
- `date()`, `time()`, `timestamp()`, `isoformat()`

---

## ⏱️ `timedelta` e aritmética

O `timedelta` representa uma diferença entre datas ou horas.

```python
from datetime import datetime, timedelta

hoje = datetime.now()
amanha = hoje + timedelta(days=1)
semana = hoje + timedelta(weeks=1)
meia_hora = hoje + timedelta(minutes=30)
saldo = hoje - timedelta(days=3, hours=5)

duracao = amanha - hoje
print(duracao.days)
print(duracao.total_seconds())
```

### Construção de durações
- `timedelta(days=1, hours=2, minutes=30)`
- `timedelta(weeks=1)`
- `timedelta(seconds=3600)`

### Operações adicionais
```python
d = timedelta(days=2)
e = d * 3
f = d / 2
print(e, f)
```

### Propriedades
- `timedelta.days`
- `timedelta.seconds`
- `timedelta.microseconds`
- `timedelta.total_seconds()`

---

## 📝 Formatação e parsing de strings

### `strftime` (datetime ➜ string)
```python
from datetime import datetime
agora = datetime.now()
print(agora.strftime('%Y-%m-%d %H:%M:%S'))
```

### `strptime` (string ➜ datetime)
```python
from datetime import datetime
texto = '27/10/2023 14:30:45'
dt = datetime.strptime(texto, '%d/%m/%Y %H:%M:%S')
```

### ISO 8601 / RFC 3339
```python
from datetime import datetime, timezone
agora = datetime.now(timezone.utc)
print(agora.isoformat())  # 2023-10-27T14:30:45.123456+00:00

parsed = datetime.fromisoformat('2023-10-27T14:30:45+00:00')
```

### Principais códigos de formatação
- `%Y`: ano com 4 dígitos
- `%m`: mês
- `%d`: dia do mês
- `%H`: hora (24h)
- `%M`: minuto
- `%S`: segundo
- `%f`: microssegundos
- `%z`: deslocamento do fuso (`+0000`)
- `%Z`: nome do fuso
- `%a` / `%A`: dia da semana abreviado/completo
- `%b` / `%B`: mês abreviado/completo

### Exemplo com timezone em parsing
```python
from datetime import datetime
texto = '2023-10-27T14:30:45-03:00'
dt = datetime.strptime(texto, '%Y-%m-%dT%H:%M:%S%z')
```

---

## 🌍 Timezones, `zoneinfo` e UTC

### Objetos `naive` vs `aware`
- `naive`: não tem informação de fuso (`tzinfo=None`)
- `aware`: inclui `tzinfo` e pode ser convertido entre fusos

```python
from datetime import datetime, timezone
naive = datetime(2023, 10, 27, 14, 30)
aware = datetime.now(timezone.utc)
```

### `timezone.utc`
```python
from datetime import datetime, timezone
utc = datetime.now(timezone.utc)
print(utc)
```

### `zoneinfo` (Python 3.9+)
```python
from datetime import datetime
from zoneinfo import ZoneInfo

brasilia = ZoneInfo('America/Sao_Paulo')
agora_sp = datetime.now(brasilia)
print(agora_sp)
```

### Converter entre fusos
```python
from datetime import datetime
from zoneinfo import ZoneInfo

utc = datetime.now(ZoneInfo('UTC'))
ny = utc.astimezone(ZoneInfo('America/New_York'))
print(ny)
```

### UTC como padrão de armazenamento
- Armazene timestamps em UTC no banco de dados.
- Converta para fusos locais apenas na apresentação.

### Exemplo completo
```python
from datetime import datetime
from zoneinfo import ZoneInfo

utc = datetime.now(ZoneInfo('UTC'))
sp = utc.astimezone(ZoneInfo('America/Sao_Paulo'))
print(sp.strftime('%Y-%m-%d %H:%M:%S %Z%z'))
```

### DST e horários ambíguos
- Use o atributo `fold` para distinguir horários duplicados no fim do horário de verão.
- `ZoneInfo` trata automaticamente da maioria dos casos.

```python
from datetime import datetime
from zoneinfo import ZoneInfo

ambiguous = datetime(2023, 11, 5, 1, 30, fold=1, tzinfo=ZoneInfo('America/New_York'))
print(ambiguous)
```

---

## 🕒 Módulo `time` e carimbos de relógio

### Medir intervalos
- `time.perf_counter()`: melhor para medir intervalos curtos.
- `time.monotonic()`: não volta no tempo se o relógio do sistema mudar.
- `time.process_time()`: tempo do processador usado pelo processo.
- `time.time()`: timestamp Unix em segundos.

```python
import time
inicio = time.perf_counter()
# ... código ...
fim = time.perf_counter()
print(fim - inicio)
```

### Sleep e agendamento simples
```python
import time

time.sleep(1.5)
```

### Conversões entre tempo local e UTC
```python
import time

print(time.time())  # segundos desde epoch
print(time.gmtime())  # UTC
print(time.localtime())  # hora local
print(time.mktime(time.localtime()))
```

### Formatar tempo com `time.strftime`
```python
import time
print(time.strftime('%Y-%m-%d %H:%M:%S', time.localtime()))
```

### Informações de fuso local
- `time.timezone`: deslocamento em segundos do UTC
- `time.altzone`: deslocamento durante horário de verão
- `time.tzname`: nomes do fuso
- `time.daylight`: indica se DST está em uso

---

## 📆 `calendar` e operações de calendário

```python
import calendar
from datetime import date

print(calendar.isleap(2024))
print(calendar.monthrange(2023, 10))  # (weekday, days no mês)
print(calendar.weekday(2023, 10, 27))
print(calendar.monthcalendar(2023, 10))
```

### Exemplo: último dia do mês
```python
import calendar
from datetime import date

ano, mes = 2023, 2
último_dia = calendar.monthrange(ano, mes)[1]
print(date(ano, mes, último_dia))
```

---

## ✅ Boas práticas para aplicações

- Armazene sempre datas/hora em UTC quando possível.
- Evite objetos `naive` em sistemas distribuídos.
- Use `datetime.fromisoformat()` e `isoformat()` para interoperabilidade.
- Prefira `zoneinfo` em vez de `pytz` se estiver usando Python 3.9+.
- Use `timedelta` para adicionar/subtrair intervalos, não versões manuais de segundos.
- Faça parsing de strings com formatos explícitos e valide entradas.

---

## 📚 Bibliotecas externas úteis

- `python-dateutil`: parsing flexível e recurrence rules.
- `pendulum`: API mais intuitiva para datas, fusos e duração.
- `arrow`: outra alternativa para manipular datas com facilidade.

### Exemplo `dateutil`
```python
from dateutil import parser

dt = parser.parse('2023-10-27T14:30:45-03:00')
print(dt)
```

---

## 📝 Resumo rápido

- `datetime` para objetos de data/hora.
- `timedelta` para durações.
- `zoneinfo` para fusos de horário nativos.
- `time` para relógio de sistema e desempenho.
- Sempre prefira UTC como referência interna.
