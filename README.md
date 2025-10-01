="SELECT '" & A2 & "' AS table_name,
 COUNT(CASE WHEN YEAR(" & B2 & ") = 2023 THEN 1 END) AS count_2023,
 COUNT(CASE WHEN YEAR(" & B2 & ") = 2024 THEN 1 END) AS count_2024,
 COUNT(CASE WHEN YEAR(" & B2 & ") = 2025 THEN 1 END) AS count_2025
FROM " & A2 & ";"
