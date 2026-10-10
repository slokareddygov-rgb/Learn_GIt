# Learn_GIt
4. Key Differences
Area	Original Original_Query	Rewritten query
Source of event data	jobdiva_events read directly	Model that already joins events to candidates
JSON parsing	Up to 5 parses of event_data per event row at query time	None at query time if the model is persisted
Join keys	JSON expressions cast to text, so no ordinary index applies	Plain columns candidate_id and job_id
Name filter	Four LOWER(...) NOT LIKE conditions	One case-insensitive regex (sriramkiran is already covered by sriram)
Denominator	qualified CTE from qualified_candidates	Same, so the percentage is unchanged
Freshness	Always live	As recent as the last model refresh
5. Where the Time Goes
Original Original_Query. The dominant cost is evaluating event_data::json ->> ... for every jobdiva_events row, in the join and again in the WHERE. The LOWER() and LIKE '%x%' filters on qualified_candidates cannot use an ordinary index, but the output is a single row. The optional created_at filter reduces the rows scanned, and helps most if that column is indexed.
Rewritten query. Cost depends on whether the model is persisted:
•	Persisted: the heavy join and JSON parsing happen once per refresh. Each dashboard load scans a flat table, so it should be faster than the original.
•	Not persisted: Metabase runs the model's SQL inline as a subquery. That includes its SELECT DISTINCT over all qc.* columns plus the full event_data text, so it is likely slower than the original.
6. Expected Outco
