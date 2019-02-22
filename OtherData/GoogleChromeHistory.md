# Google Chrome History

The `History` database (as of 2019-02-21) contains 12 tables, but no views to hint at the relationships between them.

![History Schema](schema.PNG)

The `visits`, `urls`, and `keyword_search_terms` tables can be related together to produce a browsing history that reflects web page visits and the user's search terms, if any.  The urls table records the webpage titles and locations, and it also records the number of visits to the page and the last visit date.  The visits table records each visit to the page and is related to the urls table by `visits.url = url.id`.  The `keyword_search_terms` table is related to the `url` table by `keyword_search_terms.url_id = urls.id`.  Dates are recorded in *WebKit Time*, i.e., microseconds since 1601-01-01.

The following SQLite query will result in a table that contains the following columns to the Google Chrome `History` database:

  - visit_time 
  - search_term 
  - page_title 
  - url 
  - visit_count 
  - last_visit time

### History SQL
```sql
select  
    datetime(v.visit_time  / 1000000 + (strftime('%s', '1601-01-01')), 'unixepoch', 'localtime') as visit_time, 
    k.term as search_term, 
    u.title as page_title, 
    u.url,
    u.visit_count,
    case u.last_visit_time
        when 0 then null
        else datetime(u.last_visit_time/1000000 + (strftime('%s', '1601-01-01')), 'unixepoch', 'localtime') 
    end as last_visit_time
from urls as u
left join visits as v on v.url = u.id  
left join keyword_search_terms as k on k.url_id = u.id
order by visit_time asc
```

> Note: the query does not include downloads (TBD)

The data from the above query is sorted by date, shows the search bar term as typed by the user (if applicable), and includes the total number of visits to the web page as well as the last visit date.  The format makes it simple to for the examiner to deduce from one line that "Web page X was visited 27 times between `visit_time` and `last_visit_time` in addition to documenting each visit (if still present in the database).
