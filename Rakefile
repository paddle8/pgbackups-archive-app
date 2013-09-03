# require "newrelic_rpm"
require "pgbackups-archive"
require "pony"

Pony.options = {
  :via => :smtp,
  :via_options => {
    :address => 'smtp.sendgrid.net',
    :port => '587',
    :domain => 'heroku.com',
    :user_name => ENV['SENDGRID_USERNAME'],
    :password => ENV['SENDGRID_PASSWORD'],
    :authentication => :plain,
    :enable_starttls_auto => true
  }
}

class RunPgbackupsArchive
  # include NewRelic::Agent::Instrumentation::ControllerInstrumentation

  def call
    raise "Testing PG Backup error reporting"
    Heroku::Client::PgbackupsArchive.perform
  end

  # add_transaction_tracer :call, category: :task
end

namespace :pgbackups do

  desc "Perform a pgbackups backup then archive to S3."
  task :archive do
    begin
      RunPgbackupsArchive.new.call
    rescue RuntimeError => e
      message = e.message + "\n\n" + e.backtrace.join("\n")
      Pony.mail(
        to: 'tech-notifications@archiv8.com', 
        from: 'tech-services@archiv8.com',
        subject: 'PG Backup failed',
        body: message
      )
    end
  end

end

# NewRelic::Agent.manual_start app_name: "pgbackups-archive-dummy",
#   transaction_tracer: { transaction_threshold: 1.5 }
