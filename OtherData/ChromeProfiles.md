# Google Chrome Profiles

On common task of a digital forensic examiner is to determine who has been using a computing device.  The Google Chrome browser provides a source of such information in the *Web Data* sqlite database.

I have developed a query to export the profiles created in the web browser.  I have selected those fields that were last valuable to me, but if you simply uncomment (remove the two hyphens) the line with the asterisk, you can see all the avaialbe fields.

```sql
select 
	last_name,
	first_name,
	email,
	number,
	street_address,
	city,
	state,
	zipcode,
	company_name,
	use_count,
	datetime(use_date, "unixepoch", "localtime") as used,
	datetime(date_modified, "unixepoch", "localtime") as modified
	-- , * 
from autofill_profiles as p
left join autofill_profile_names as n on p.guid = n.guid
left join autofill_profile_emails as e on p.guid = e.guid
left join autofill_profile_phones as ph on p.guid = ph.guid
```

I have not done any testing as to the *use_date* meaning, but bear in mind that Chrome users can login to a browser and remain logged in.  I suspect the *use_date* represents the last date on which the profile was logged into, not the last date the profile was acitvely in use.
