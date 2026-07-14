require "contentful"
require "yaml"
require "fileutils"
require "date"

# Refreshes the committed Contentful snapshot that the site builds from.
#
#   CONTENTFUL_SPACE_ID=xxx CONTENTFUL_ACCESS_TOKEN=yyy bundle exec rake contentful:import
#
# CONTENTFUL_ACCESS_TOKEN is a read-only Content Delivery API token.
# This is a manual, occasional step: run it, then commit the updated YAML.
# It is NOT part of the build/deploy — _data/contentful/spaces/acparts.yaml
# is checked in, so GitHub Actions builds the site without touching Contentful.
#
# Output format mirrors the old jekyll-contentful-data-import gem so the
# Liquid templates (site.data.contentful.spaces.acparts.<type>) keep working:
# one top-level key per content type, each an array of entries carrying a
# `sys` block plus flattened fields; assets resolve to a hash with `url`,
# linked entries to `{ sys: { id: ... } }`.

SPACE_NAME = "acparts".freeze
OUTPUT = File.join(__dir__, "_data", "contentful", "spaces", "#{SPACE_NAME}.yaml").freeze

namespace :contentful do
  desc "Import Contentful entries into _data/contentful/spaces/#{SPACE_NAME}.yaml"
  task :import do
    client = Contentful::Client.new(
      space: fetch_env("CONTENTFUL_SPACE_ID"),
      access_token: fetch_env("CONTENTFUL_ACCESS_TOKEN"),
      environment: ENV.fetch("CONTENTFUL_ENVIRONMENT", "master"),
      dynamic_entries: :auto,
      raise_errors: true
    )

    entries = fetch_all_entries(client)

    grouped = Hash.new { |h, k| h[k] = [] }
    entries.each { |entry| grouped[entry.content_type.id] << serialize_entry(entry) }

    # Stable content-type ordering; entry order within a type is preserved.
    data = grouped.keys.sort.each_with_object({}) { |k, acc| acc[k] = grouped[k] }

    FileUtils.mkdir_p(File.dirname(OUTPUT))
    File.write(OUTPUT, data.to_yaml)
    puts "Wrote #{entries.size} entries across #{grouped.size} content types to #{OUTPUT}"
  end
end

def fetch_env(key)
  ENV[key] || abort("Missing #{key}. Set it before running (Content Delivery API credentials).")
end

# Contentful returns at most 100 entries per request; page through them all.
def fetch_all_entries(client)
  all = []
  skip = 0
  limit = 100
  loop do
    page = client.entries(limit: limit, skip: skip, include: 2, order: "sys.createdAt")
    all.concat(page.to_a)
    break if all.length >= page.total
    skip += limit
  end
  all
end

def serialize_entry(entry)
  hash = {
    "sys" => {
      "id" => entry.id,
      "created_at" => iso(entry.created_at),
      "updated_at" => iso(entry.updated_at),
      "content_type_id" => entry.content_type.id,
      "revision" => entry.revision
    }
  }
  entry.fields.each { |name, value| hash[name.to_s] = serialize_value(value) }
  hash
end

def serialize_value(value)
  case value
  when Contentful::Asset
    {
      "sys" => { "id" => value.id, "created_at" => iso(value.created_at), "updated_at" => iso(value.updated_at) },
      "title" => value.title,
      "description" => value.description,
      "url" => value.file&.url
    }
  when Contentful::Entry
    { "sys" => { "id" => value.id, "content_type_id" => value.content_type.id } }
  when Array
    value.map { |v| serialize_value(v) }
  when DateTime, Time
    iso(value)
  else
    value
  end
end

def iso(time)
  time&.iso8601
end
