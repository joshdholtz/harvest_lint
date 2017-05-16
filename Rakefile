desc "Gets a CSV with data from an event"
task :spec, [:yaml_path, :page_size, :meta_fields] do |t, args|

  require 'dotenv'
  require 'harvested'

  Dotenv.load
  
  # Get yaml configuration
  yaml_path = args[:yaml_path]
  raise "yaml_path is required" unless yaml_path
  yaml =  YAML.load_file(yaml_path)
  
  # Parse yaml config
  @config_start_str = yaml["start_date"]
  @config_end_str = yaml["end_date"]
  @config_start = Time.parse(@config_start_str)
  @config_end = Time.parse(@config_end_str)
  @config_harvest = yaml["harvest"] || {}
  @config_reports = yaml["reports"] || {}
  @config_active_projects_without_budget = @config_reports["active_projects_without_budget"]
  @config_clients = @config_reports["clients"].map do |client| 
    [client["id"], client]
  end.to_h
  @config_projects = @config_reports["projects"].map do |project| 
    [project["id"], project]
  end.to_h
  
  @project_reports = {}
  def add_project_report(project_id, key, value)
    project_report = @project_reports[project_id] || {}
    project_report[key] = value
    @project_reports[project_id] = project_report
  end
  
  def report_no_notes(config, harvest, project, enteries, users)
    if config["no_notes"]
      empty_notes = enteries.select do |entry|
        entry.notes.to_s == ""
      end 
      empty_note_users = enteries.map do |entry|
        users[entry.user_id]
      end.uniq
      
      if !empty_notes.empty?
        add_project_report(project.id, :empty_note_count, empty_notes.count)
        add_project_report(project.id, :empty_note_users, empty_note_users)
      end
    end
  end

  def report_low_hours(config, harvest, project, enteries, users)
    low_hours = config["low_hours"]
    if low_hours
      total_hours = enteries.map{|e|e.hours}.inject(0){|sum,x| sum + x }
      add_project_report(project.id, :low_hours, total_hours) if total_hours <= low_hours
    end
  end

  def report_high_hours(config, harvest, project, enteries, users)
    high_hours = config["high_hours"]
    if high_hours
      total_hours = enteries.map{|e|e.hours}.inject(0){|sum,x| sum + x }
      add_project_report(project.id, :high_hours, total_hours) if total_hours >= high_hours
    end
  end

  def report_active_projects_without_budges(harvest, project)
    add_project_report(project.id, :no_budget, true) if project.active && project.estimate.to_i <= 0
  end
  
  def run
    # Init harvest client
    subdomain = ENV[@config_harvest["subdomain_env"]] if @config_harvest["subdomain_env"]
    username = ENV[@config_harvest["username_env"]] if @config_harvest["username_env"]
    password = ENV[@config_harvest["password_env"]] if @config_harvest["password_env"]
    subdomain = @config_harvest["subdomain"] unless subdomain
    username = @config_harvest["username"] unless username
    password = @config_harvest["password"] unless password
    harvest = Harvest.client(subdomain: subdomain, username: username, password: password)
    
    # Map all users to ids
    users = harvest.users.all.map do |user| 
      [user.id, user]
    end.to_h
    
    # Fetch all projects
    projects = harvest.projects.all.map do |project| 
      [project.id, project]
    end.to_h
    
    projects.values.each do |project|
      report_active_projects_without_budges(harvest, project) if @config_active_projects_without_budget
      
      config = @config_clients[project.client_id] || @config_projects[project.id]
      if config
        enteries = harvest.reports.time_by_project(project, @config_start, @config_end)
        
        report_no_notes(config, harvest, project, enteries, users)
        report_low_hours(config, harvest, project, enteries, users)
        report_high_hours(config, harvest, project, enteries, users)
      end
    end
    
    make_report(:console, @config_start_str, @config_end_str, projects, @project_reports)
  end
  
  run
end

def make_report(type, config_start, config_end, projects, project_reports)
  print_console(config_start, config_end,projects, project_reports) if type == :console
end

def print_console(config_start, config_end, projects, project_reports)
  puts ""
  puts "========================================================================"
  puts "Report for #{config_start} to #{config_end}" 
  puts "========================================================================"
  puts ""
  
  @project_reports.each do |project_id, report|
    project = projects[project_id]
    puts "=== #{project.name} ==="
    
    report.keys.sort.each do |key|
      value = report[key]
      if value.is_a? Array
        value = value.map do |a_value|
          if a_value.is_a? Harvest::User
            a_value = "#{a_value.first_name} #{a_value.last_name} <#{a_value.email}>"
          end
        end.join(", ")
      elsif value.is_a? Numeric
        value = value.round(2)
      end
      
      puts "\t#{key}: #{value}"
    end
    
  end
end
