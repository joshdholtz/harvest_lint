# harvest_lint

Get warnings of bad project state in Harvest

## Sample config

```yml
start_date: "05/01/2017" # Start date of time entries to query
end_date: "05/08/2017" # End date of time entries to query
harvest:
  subdomain: your_subdomain
  username: your_username
  password: your_password
reports:
  active_projects_without_budget: true # Reports projects without a budget
  team:
    - id: <some_user_id>
      low_hours: 10 # Reports if hours below this
      high_hours: 30 # Reports if hours above this
      no_notes: true # Reports number of time entries without notes
    - id: <some_other_user_id>
      low_hours: 10
      high_hours: 20
      no_notes: true
  clients:
    - id: <some_client_id>
      low_hours: 10 # Reports if hours below this
      high_hours: 30 # Reports if hours above this
      no_notes: true # Reports number of time entries without notes (and who all had them)
  projects:
    - id: <some_project_id_id>  
      low_hours: 10 # Reports if hours below this
      high_hours: 15 # Reports if hours above this
      no_notes: false # Won't report empty notes in time entries (default value)
```
