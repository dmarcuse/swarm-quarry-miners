local config = {}
local current_file = nil

function init(file)
	if file then
		if fs.exists(file) then
			local f = fs.open(file, "r")
			local data = f.readAll()
			f.close()
			local c = textutils.unserialize(data)
			if c then config = c else config = {} end
			current_file = file
		else
			config = {}
			current_file = file
		end
	else
		config = {}
		current_file = nil
	end
end

local function save()
	if current_file then
		local f = fs.open(current_file, "w")
		f.write(textutils.serialize(config))
		f.close()
	end
end

function get(key, default)
	return config[key] or default
end

function set(key, value)
	config[key] = value
	save()
end