- Grant permission to users in metabase: 
	- Query:
		GRANT USAGE ON SCHEMA biz TO cos_user;
		GRANT SELECT ON ALL TABLES IN SCHEMA biz TO cos_user;
	- Create new database on metabase -> add user