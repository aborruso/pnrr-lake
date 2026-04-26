# pnrr-lake

Frozen DuckLake con i dati open del PNRR italiano, interrogabile da chiunque
via DuckDB senza server, senza registrazione, senza API key.

Fonte: [italiadomani.gov.it](https://www.italiadomani.gov.it/content/sogei-ng/it/it/catalogo-open-data.html)
— aggiornamento automatico dal sito ufficiale.

---

## Tabelle disponibili

| Tabella | Righe | Descrizione |
|---|---|---|
| `progetti` | 280.769 | Tutti i progetti PNRR con dettagli finanziari e stato avanzamento |
| `localizzazione` | 335.629 | Localizzazione geografica dei progetti (regione, provincia, comune) |

Le due tabelle si collegano tramite `CUP` e `"Codice Locale Progetto"`.

---

## Come usarlo

Installa DuckDB se non ce l'hai: [duckdb.org/docs/installation](https://duckdb.org/docs/installation/)

```bash
duckdb << 'EOF'
LOAD ducklake;
LOAD httpfs;
ATTACH 'ducklake:https://raw.githubusercontent.com/aborruso/pnrr-lake/main/lake.ducklake' AS lake;
USE lake;

SHOW TABLES;
SELECT COUNT(*) FROM progetti;
EOF
```

---

## Query di esempio

```sql
-- finanziamento PNRR per missione (miliardi €)
SELECT Missione, "Descrizione Missione",
  ROUND(SUM("Finanziamento PNRR") / 1e9, 2) AS miliardi_eur
FROM progetti
GROUP BY ALL
ORDER BY miliardi_eur DESC;

-- stato avanzamento
SELECT "Stato Avanzamento Progetto", COUNT(*) AS n
FROM progetti
GROUP BY 1 ORDER BY n DESC;

-- progetti per regione
SELECT "Descrizione Regione", COUNT(DISTINCT CUP) AS n_progetti
FROM localizzazione
WHERE "Descrizione Regione" != 'AMBITO NAZIONALE'
GROUP BY 1 ORDER BY n_progetti DESC;

-- join: finanziamento medio per regione
SELECT l."Descrizione Regione",
  COUNT(DISTINCT p.CUP) AS n_progetti,
  ROUND(AVG(p."Finanziamento PNRR") / 1000, 1) AS media_k_eur
FROM progetti p
JOIN localizzazione l ON p.CUP = l.CUP
WHERE l."Descrizione Regione" != 'AMBITO NAZIONALE'
GROUP BY 1 ORDER BY media_k_eur DESC;
```

---

## Aggiornare il lake

```bash
./scripts/build.sh
git add lake.ducklake dati/
git commit -m "aggiornamento dati $(date +%Y-%m-%d)"
git push
```
