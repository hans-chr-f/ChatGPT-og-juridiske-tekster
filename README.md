# ChatGPT -- et førsteinntrykk
Dette er en side for supplerende material til artikkelen *ChatGPT. Et førsteinntrykk*, trykt i [Lov og Rett nr.~X, 2023][1]

## Common Crawl
Common Crawl er den største kilden til GPT-3s treningsgrunnlag. Common Crawl arkiverer store deler av nettet og gjør arkivene tilgjengelig for alle. Her skal vi undersøke hvilke norske domener som er en del av Common Crawl. Hensikten er å kaste lys over hvilke domener som *kan* være en del av treningsgrunnlaget til GPT-3. Her dokumenterer vi framgangsmåten.

Common Crawl er fritt tilgjengelig på Amazon~S3. Men filene er veldig store og vanskelig å analysere direkte. Vi har derfor valgt å bruke AWS Athena som lar oss analysere dataene på en ekstern server ved hjelp av SQL. Vi fulgte framgangsmåten [her][2].

Først opprettet vi en database:
```
CREATE DATABASE ccindex
```

Deretter opprettet vi en tabell i databasen:
```
CREATE EXTERNAL TABLE IF NOT EXISTS ccindex (
  url_surtkey                   STRING,
  url                           STRING,
  url_host_name                 STRING,
  url_host_tld                  STRING,
  url_host_2nd_last_part        STRING,
  url_host_3rd_last_part        STRING,
  url_host_4th_last_part        STRING,
  url_host_5th_last_part        STRING,
  url_host_registry_suffix      STRING,
  url_host_registered_domain    STRING,
  url_host_private_suffix       STRING,
  url_host_private_domain       STRING,
  url_protocol                  STRING,
  url_port                      INT,
  url_path                      STRING,
  url_query                     STRING,
  fetch_time                    TIMESTAMP,
  fetch_status                  SMALLINT,
  content_digest                STRING,
  content_mime_type             STRING,
  content_mime_detected         STRING,
  content_charset               STRING,
  content_languages             STRING,
  warc_filename                 STRING,
  warc_record_offset            INT,
  warc_record_length            INT,
  warc_segment                  STRING)
PARTITIONED BY (
  crawl                         STRING,
  subset                        STRING)
STORED AS parquet
LOCATION 's3://commoncrawl/cc-index/table/cc-main/warc/';
```

Så kjørte vi følgende kommando (jf. veiledningen):
```
MSCK REPAIR TABLE ccindex`
```

Da skulle alt være klart til å kjøre den første spørringen:
```
SELECT COUNT(*) AS count, url_host_registered_domain
FROM "ccindex"."ccindex"
WHERE crawl = 'CC-MAIN-2018-05'
  AND subset = 'warc'
  AND url_host_tld = 'no'
GROUP BY  url_host_registered_domain
HAVING (COUNT(*) >= 100)
ORDER BY  count DESC
```
Denne spørringen returnerer antall sider (count) for hvert domene som slutter på '.no'. Den begrenser til arkivet for uke 5 i 2018 (CC-MAIN-2018-05) og tar bare med domener med 100 eller flere sider arkivert.

For å rekonstruere grunnlaget GPT-3 trenger vi 41 slike arkiver. For perioden 2016-2019 har Common Crawl 45 arkiver. OpenAI har ikke eksplisitt sagt hvilke arkiver de bruker. Men ut fra publiseringsdatoer og hvis vi regner med at det tar en del tid både å klargjøre datasett og å trene modellene virker det rimelig å anta at det er de fire siste arkivene i 2019 som ikke er med.

Følgende spørring gir oss en oversikt over antall sider pr. domene for alle 41 arkiver. Vi begrenser oss til domener som slutter på '.no'.

```
SELECT COUNT(*) AS count,
   url_host_registered_domain
FROM "ccindex"."ccindex"
WHERE REGEXP_LIKE(crawl, '^CC-MAIN-201([6-8]|9-([0-2]|3[05]))') 
  AND subset = 'warc' 
   AND url_host_tld = 'no' 
GROUP BY url_host_registered_domain 
HAVING (COUNT(*) >= 100) 
ORDER BY count DESC
```

Resultatene ligger på en egen side [her][3]


## WebText


[1]: https://www.idunn.no/journal/lor
[2]: https://commoncrawl.org/2018/03/index-to-warc-files-and-urls-in-columnar-format/
[3]: https://github.com/hans-chr-f/ChatGPT.-Et-f-rsteinntrykk/blob/main/Common_crawl_norske_domener.csv
