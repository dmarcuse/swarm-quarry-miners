--[[ relgo
A modified version of the 'go' API - functionally identical, but all coordinates are relative to where they were initially set. No GPS required, however, it may be less versatile.

Written by apemanzilla
]]

local pos = {}

local df = {
	[0] = {		-- South
		dx = 0,
		dz = 1
	},
	[1] = {		-- West
		dx = -1,
		dz = 0
	},
	[2] = {		-- North
		dx = 0,
		dz = -1
	},
	[3] = {		-- East
		dx = 1,
		dz = 0
	}
}

--[[ Utility Functions ]]

local function dirToFacing(dir)
	if dir == "north" then
		return 2
	elseif dir == "south" then
		return 0
	elseif dir == "east" then
		return 3
	elseif dir == "west" then
		return 1
	end
end

function savePos()
	if pos.x and pos.y and pos.z and pos.f then
		local f = fs.open(".position","w")
		f.write(textutils.serialize(pos))
		f.close()
	end
end

function loadPos()
	if fs.exists(".position") then
		local f = fs.open(".position","r")
		local temppos = textutils.unserialize(f.readAll())
		f.close()
		if (temppos.x and temppos.y and temppos.z and temppos.f) then
			pos.x, pos.y, pos.z, pos.f = temppos.x, temppos.y, temppos.z, temppos.f
		else
			pos.x = 0
			pos.y = 64
			pos.z = 0
			pos.f = 0
		end
	else
		pos.x = 0
		pos.y = 64
		pos.z = 0
		pos.f = 0
	end
end

loadPos()

function getPos()
	return pos
end

function setPos(x,y,z,f)
	assert(x and y and z and f,"expected number, number, number, number")
	pos.x = x
	pos.y = y
	pos.z = z
	pos.f = f
	savePos()
end

function adjustCoords(tbl, change, f)
	assert(tbl.x and tbl.y and tbl.z, "missing coordinate(s)")
	tbl.x = tbl.x + (df[f].dx * change)
	tbl.z = tbl.z + (df[f].dz * change)
	return tbl
end

local turtle = {}
for k,v in pairs(_G.turtle) do
	turtle[k] = v
end

function fuelTo(x, y, z, fromPos)
	if type(x) == "table" then
		y = x.y
		z = x.z
		f = x.f
		x = x.x
	end
	local p = fromPos or pos
	local needed = math.abs(p.x - x) + math.abs(p.y - y) + math.abs(p.z - z)
	return needed
end

function canReach(x, y, z)
	if type(x) == "table" then
		y = x.y
		z = x.z
		f = x.f
		x = x.x
	end
	if not (pos.x and pos.y and pos.z) then
		return false, "cannot determine position"
	end
	if not x then x = pos.x end
	if not y then y = pos.y end
	if not z then z = pos.z end
	local fuelNeeded = math.abs(pos.x - x) + math.abs(pos.y - y) + math.abs(pos.z - z)
	if turtle.getFuelLevel() < fuelNeeded then
		return false, fuelNeeded
	end
	return true, fuelNeeded
end

function pE(p1, p2)
	return p1.x == p2.x and p1.y == p2.y and p1.z == p2.z
end

--[[ Basic Movement Functions ]]

function forward()
	local success, err = turtle.forward()
	if success then
		if pos.x and pos.z and pos.f then
			pos.x = pos.x + df[pos.f].dx
			pos.z = pos.z + df[pos.f].dz
		end
		savePos()
		return true
	else
		return false, err
	end
end

function back()
	local success, err = turtle.back()
	if success then
		if pos.x and pos.z and pos.f then
			pos.x = pos.x - df[pos.f].dx
			pos.z = pos.z - df[pos.f].dz
		end
		savePos()
		return true
	else
		return false, err
	end
end

function up()
	local success, err = turtle.up()
	if success then
		if pos.y then
			pos.y = pos.y + 1
		end
		savePos()
		return true
	else
		return false, err
	end
end

function down()
	local success, err = turtle.down()
	if success then
		if pos.y then
			pos.y = pos.y - 1
		end
		savePos()
		return true
	else
		return false, err
	end
end

function right()
	local success, err = turtle.turnRight()
	if success then
		if pos.f then
			pos.f = pos.f + 1
			if pos.f > 3 then pos.f = 0 end
		end
		savePos()
		return true
	else
		return false, err4
	end
end

function left()
	local success, err = turtle.turnLeft()
	if success then
		if pos.f then
			pos.f = pos.f - 1
			if pos.f < 0 then pos.f = 3 end
		end
		savePos()
		return true
	else
		return false, err
	end
end

--[[ Advanced Movement Functions ]]

function rotateTo(f)
	if f > 3 or f < 0 then
		return false, "invalid facing"
	end
	if not pos.f then
		return false, "cannot determine facing"
	end
	local function simulateRotateTo(startf, endf)
		local right, left = 0, 0
		local f = startf
		while f ~= endf do
			right = right + 1
			f = f + 1
			if f > 3 then f = 0 end
		end
		f = startf
		while f ~= endf do
			left = left + 1
			f = f - 1
			if f < 0 then f = 3 end
		end
		return right, left
	end
	local r,l = simulateRotateTo(pos.f, f)
	if r <= l then
		for i = 1, r do
			right()
		end
	else
		for i = 1, l do
			left()
		end
	end
	return true
end

function goto(x, y, z, f)
	if type(x) == "table" then
		y = x.y
		z = x.z
		f = x.f
		x = x.x
	end
	-- Validation and inital setup
	if f and (f < 0 or f > 3) then
		return false, "invalid facing"
	end
	if not (pos.x and pos.y and pos.z and pos.f) then
		local success, err = updatePos()
		if not success then return false, err end
	end
	local start = {}
	start.x, start.y, start.z, start.f = pos.x, pos.y, pos.z, pos.f
	local dest = {}
	dest.x, dest.y, dest.z, dest.f = x, y, z, f
	if not dest.x then dest.x = start.x end
	if not dest.y then dest.y = start.y end
	if not dest.z then dest.z = start.z end
	if not dest.f then dest.f = start.f end

	if canReach(dest.x, dest.y, dest.z) then
		-- Function to determine facing necessary for coordinate change
		local function facingForCoordChange(axis, change)
			if axis == "x" then
				if change > 0 then
					return 3
				elseif change < 0 then
					return 1
				else
					return
				end
			elseif axis == "z" then
				if change > 0 then
					return 0
				elseif change < 0 then
					return 2
				else
					return
				end
			end
			return
		end
		-- Move to specified coordinates
		-- Y up
		if start.y < dest.y then
			for i = 1, math.abs(start.y - dest.y) do
				while not up() do
					while peripheral.getType("top") == "turtle" do sleep(1) end
					turtle.digUp()
					turtle.attackUp()
				end
			end
		end
		-- Y down
		if start.y > dest.y then
			for i = 1, math.abs(start.y - dest.y) do
				while not down() do
					while peripheral.getType("bottom") == "turtle" do sleep(1) end
					turtle.digDown()
					turtle.attackDown()
				end
			end
		end
		-- X
		if facingForCoordChange("x", dest.x - start.x) then
			rotateTo(facingForCoordChange("x", dest.x - start.x))
		end
		for i = 1, math.abs(start.x - dest.x) do
			while not forward() do
				--while peripheral.getType("front") == "turtle" do sleep(1) end
				turtle.dig()
				turtle.attack()
			end
		end
		-- Z
		if facingForCoordChange("z", dest.z - start.z) then
			rotateTo(facingForCoordChange("z", dest.z - start.z))
		end
		for i = 1, math.abs(start.z - dest.z) do
			while not forward() do
				--while peripheral.getType("front") == "turtle" do sleep(1) end
				turtle.dig()
				turtle.attack()
			end
		end
		-- Rotate
		rotateTo(dest.f)
		-- Attempt to validate position
		local final = {}
		final.x, final.y, final.z = gps.locate(0.1)
		if final.x and final.y and final.z and validatePos then
			if final.x == dest.x and final.y == dest.y and final.z == dest.z then
				return true
			else
				return false, "final position does not match"
			end
		else
			-- Assume we are in the right spot
			return true
		end
	else
		return canReach(dest.x, dest.y, dest.z)
	end
end