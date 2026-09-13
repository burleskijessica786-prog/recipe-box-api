2026-09-13 – GET /recipes → 200, returned full recipe list to an anonymous requester
2026-09-13 – POST /recipes → 201, created a new recipe from an anonymous requester
2026-09-13 – DELETE /recipes/<4> → 200, deleted an existing recipe as an anonymous requester; later GET /recipes/<4> returned 404 (record gone)