# Rutina — continuar en Cowork / Claude Code

App de seguimiento de cronograma académico. Cronograma completo día por día, marcado de completo/parcial, sistema de deuda de pomodoros y carga semanal ajustable.

## Estado actual

Funcionando y probado. Falta un solo paso: **publicarla en GitHub Pages**.

## Archivos

```
index.html      La app entera (HTML + CSS + JS, sin dependencias)
sw.js           Service worker (funciona sin internet)
manifest.json   Config PWA (instalable en el celular)
icon-192.png    Ícono
icon-512.png    Ícono grande
```

Todo en la raíz. `index.html` tiene que llamarse así para que GitHub Pages lo sirva.

---

## Lo que hay que pedirle al agente

Copiá y pegá esto:

> Tengo estos archivos en una carpeta. Creá un repo público en GitHub llamado `rutina`, subí los cinco archivos a la raíz, activá GitHub Pages desde la branch main, y pasame la URL final. Verificá que el service worker esté sirviéndose bien y que el manifest cargue sin errores.

El agente necesita `git` instalado y tu cuenta de GitHub autenticada (por `gh auth login` o token). Si no lo está, te va a guiar.

---

## Contexto del proyecto

### Cronograma base
Cursada de Economía en UNR, segundo cuatrimestre 2026, arranca el **3 de agosto**:

| Día | Materias |
|---|---|
| Lunes | Econometría I (Aula P) 15–17 |
| Martes | Monetaria (E) 13–15 · Econometría (K) 15–17 · Macro II (K) 19–21 |
| Miércoles | Monetaria (C) 13–15 · Regional (11) 17–19 |
| Jueves | Econometría virtual 15–17 · Macro II virtual 19–21 |
| Viernes | Regional virtual 17–19 |

Además: gimnasio 07:30 lun/mar y 08:45 vie, club de rugby jueves 21–24, kinesiología todos los días menos martes, psicólogo miércoles 08:00 cada 15 días.

### Lógica de la app

**Marcado:** cada bloque tiene ✓ (completo) y ½ (parcial, cuenta 0,5).

**Carga semanal** — escala cuántos pomodoros se esperan por día:
- Tranquila: 16/semana
- Normal: 28/semana
- Parcial cerca: 37/semana

**Deuda:** pomodoros no hechos de lunes a viernes se acumulan y aparecen el fin de semana como bloques extra, con techo de 4 por día. Si la deuda pasa de 8, la app avisa que la carga fue demasiado alta.

**Adelantar horas:** botón `+` en la tarjeta "Pomodoros hoy" (solo lun–vie). Cada pomodoro adelantado primero cancela deuda de la semana; lo que sobra reduce la meta del sábado y domingo (se descuenta primero del sábado, luego del domingo). Guardado en `S.adel` por fecha.

**Super semana:** si el total de pomodoros hechos en la semana supera la meta de la carga elegida (16/28/37 según modo), aparece una insignia 🔥 en Hoy y en Semana.

**Temporizador integrado:** botón flotante `▶` abajo a la derecha. Timer de 50 min de foco + 10 de descanso, alternando (constantes `FOCO`/`DESC`). Al terminar un bloque de foco, `completarPomodoro()` marca como ✓ el próximo bloque de estudio libre del día; si no quedan, suma un adelantado. El estado corre en memoria (objeto `T`) y usa timestamp de fin para ser preciso aunque se apague la pantalla.

**Datos:** todo en `localStorage` bajo la clave `rutina_v3`. Nada sale del dispositivo.

### Estructura del código

`index.html` es un solo archivo. Las partes principales:

- `PLAN` — objeto con los bloques de cada día (0=domingo a 6=sábado)
- `MODES` — los tres niveles de carga con su factor multiplicador
- `bloques(d)` — devuelve los bloques del día ajustados por carga
- `deuda()` — deuda neta = deuda bruta menos lo adelantado en la semana
- `adelSemana()` / `adelSobrante()` — pomodoros adelantados y sobrante tras cancelar deuda
- `esperaFinde(d)` — meta del sábado/domingo ya reducida por el adelanto sobrante
- `superSemana()` — true si el total hecho supera la meta de la carga
- `pctDia(d)` — porcentaje de cumplimiento de un día

Para cambiar el cronograma, editás `PLAN`. Cada bloque:
```js
{id:'l-p1', t:'Pomodoro 1', h:'09:10–10:00', k:'pom'}
```
`k` puede ser: `pom`, `class`, `gym`, `kine`, `read`, `soc`, `psi`, `rest`, `plan`.

---

## Cosas que se pueden mejorar después

- Sincronización entre dispositivos (hoy los datos viven solo en el celu)
- Notificaciones push nativas (en iOS son poco confiables; por ahora se usan los recordatorios del teléfono)
- Ajustar el cronograma cuando cambien los horarios de cursada
- Exportar el historial a CSV para analizarlo

---

## Recordatorios ya cargados

En la app Recordatorios del teléfono, arrancan el 2-3 de agosto:

| Cuándo | Qué |
|---|---|
| Dom 20:00 | Planificar la semana |
| Dom-jue 21:30 | Preparar apuntes + ropa de gym |
| Lun-mié 09:10 | Estudio profundo |
| Diario 12:15 | Lectura no económica |
| Sáb 10:00 | Estudio 4 horas |

Estos son independientes de la app.
