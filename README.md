[//]: # (To view this file use: python -m pip install grip; python -m grip -b "README.md")
[//]: # (https://github.com/settings/tokens)
[//]: # (vim ~/.grip/settings.py)
[//]: # (PASSWORD = 'YOUR-ACCESS-TOKEN')
[//]: # (https://github.com/naokazuterada/MarkdownTOC)

# README
Create a .env file like this:
```
MIDEA_EMAIL=youremail@yourdomain.com
MIDEA_PASSWORD=secretpass
MIDEA_APP_KEY=3742e9e5842d4ad59c2db887e12449f9
```

## Some tests
```ruby
require "midea_air_condition"

client = MideaAirCondition::Client.new(
  ENV["MIDEA_EMAIL"],
  ENV["MIDEA_PASSWORD"],
  app_key: ENV["MIDEA_APP_KEY"]
)

client.debug = true
client.login
devices = client.appliance_list

target = 0

devices.each do |device|
  print "[id=#{device['id']} type=#{device['type']}]"
  print " #{device['name']} is "
  print 'not ' if device['onlineStatus'] != '1'
  print 'online and '
  print 'not ' if device['activeStatus'] != '1'
  puts 'active.'

  target = device['id'] if device['onlineStatus'] == '1'
end

# Status
builder = client.new_packet_builder
command = MideaAirCondition::Command::RequestStatus.new
builder.add_command(command)

response = client.appliance_transparent_send(
  target,
  builder.finalize
)

device = MideaAirCondition::Device.new(response)

def show_status(target, device)
  puts "#{target} is turned #{(device.power_status ? 'on' : 'off')}."
  puts 'Details:'
  puts "  Target temperature is #{device.temperature} celsius."
  puts "  // Indoor temperature: #{device.indoor_temperature} celsius."
  puts "  // Outdoor temperature: #{device.outdoor_temperature} celsius."
  puts "  Mode: #{device.mode_human}."
  puts "  Fan speed: #{device.fan_speed}."
  puts "  TimerOn is #{(device.on_timer[:status] ? '' : 'not')} active."
  puts "    at:  #{device.on_timer_human}" if device.on_timer[:status]
  puts "  TimerOff is #{(device.off_timer[:status] ? '' : 'not')} active."
  puts "    at: #{device.off_timer_human}" if device.off_timer[:status]
  puts "  Eco mode is #{(device.eco ? 'on' : 'off')}."
end
show_status(target, device)

(device.data[0x33 + 10] - 50) / 2

# Turn on
builder = client.new_packet_builder
command = MideaAirCondition::Command::Set.new
command.turn_on
command.temperature 25
builder.add_command(command)

response = client.appliance_transparent_send(
  target,
  builder.finalize
)

device = MideaAirCondition::Device.new(response)
puts "  Target temperature is #{device.temperature} celsius."

# Turn off
builder = client.new_packet_builder
command = MideaAirCondition::Command::Set.new
command.turn_off
builder.add_command(command)

response = client.appliance_transparent_send(
  target,
  builder.finalize
)
device = MideaAirCondition::Device.new(response)
puts "  Target temperature is #{device.temperature} celsius."

# Set temperature to 23 celsius
builder = client.new_packet_builder
command = MideaAirCondition::Command::Set.new
command.temperature 23
builder.add_command(command)

response = client.appliance_transparent_send(
  target,
  builder.finalize
)

device = MideaAirCondition::Device.new(response)
puts "  Target temperature is #{device.temperature} celsius."
```