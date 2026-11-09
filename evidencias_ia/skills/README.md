# Desarrollo de las Skills

En [`../conversacion_desarrollo.md`](../conversacion_desarrollo.md), buscar «limpiando-enaho», «graficando-enaho» y
«mapeando-enaho»: escritura de los scripts y `SKILL.md`, la validación (`salidas/validacion.txt`, 15/16 aprobadas),
y las correcciones:

- `mieperho` llega como int8 y desborda en `12 × mieperho` → conversión a decimal;
- la consola de Windows no imprimía UTF-8 → `sys.stdout.reconfigure`;
- falsa alarma revisada: en `sumaria-2024.dta` las variables `gru1*` son solo alimentos (los grupos 10 a 12 están
  en `sumaria-2024-12g.dta`);
- la suma de los grupos de gasto no es `gashog2d` → la validación se cambió por «alimentos entre 0 y el gasto total»
  (19 hogares fuera, se avisa);
- títulos largos cortados en los gráficos → `textwrap`.

## Coordinación entre Skills (prueba)

En una copia limpia del repositorio (`git archive` del commit `092de75`, con solo los zip del INEI y el GeoJSON en
`datos/`, sin `salidas/` ni `mapas/`) se hizo un pedido en lenguaje natural, sin nombrar ninguna Skill:

```bash
claude -p "Haz un mapa de la pobreza monetaria por departamento con ENAHO 2024." --model opus \
  --mcp-config .mcp.json --strict-mcp-config --allowedTools "mcp__bcrp,Skill,Bash,Read,Write,Edit,Glob,Grep" \
  --disallowedTools "WebSearch,WebFetch" --output-format stream-json --verbose > coordinacion_skills.jsonl
```

Registro legible: [`coordinacion_skills.md`](coordinacion_skills.md) (crudo en `coordinacion_skills.jsonl`, con las
rutas del perfil de Windows ocultas).

| Paso | Qué hizo el agente |
|---|---|
| 1 | Activó `mapeando-enaho` por el pedido (mapa por departamento) |
| 2 | Revisó las entradas y vio que no existía `salidas/hogares_2024.parquet` |
| 3 | Activó `limpiando-enaho`, como manda el paso 1 de `mapeando-enaho`, y corrió `limpiar.py` desde los zip |
| 4 | Leyó la validación: `VÁLIDA` (15 controles aprobados y 1 aviso); pobreza 27,58 %, 108 664 residentes, 25 departamentos |
| 5 | Corrió `mapear.py`: 25/25 departamentos unidos por código, Lima y Callao separados, diferencia máxima con la referencia de 0,05 pp |
| 6 | Entregó el mapa, los cinco departamentos con más y con menos pobreza y la fuente |

Duró 53 segundos (11 turnos). El mapa que salió es idéntico (mismo MD5) a
[`mapas/pobreza_departamento_2024.png`](../../mapas/pobreza_departamento_2024.png), y `salidas/indicadores.csv` y
`salidas/comparacion_mapa.csv` coinciden con los publicados. Dos intentos con `PowerShell` fueron rechazados porque
esa herramienta no estaba permitida; el agente siguió con `Bash`.
