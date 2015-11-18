assert(turtle, "This api is for turtles only")
assert(go, "This api requires the 'go' api")
assert(config, "This api requires the 'config' api")


-- list of junk blocks to avoid mining and toss if necessary
local blacklist = config.get("block_blacklist")

-- list of liquids that can be used as fuel
local liquidfuels = {
	lava = {name="minecraft:flowing_lava",damage=0}
}

-- converts block-structured data to item-structured data
function b2i(data)
	if data then
		local out = {}
		out.name = data.name
		out.damage = data.damage or data.metadata
		out.count = data.count or 1
		return out
	end
end

-- returns true if a block is "junk", false otherwise
function isJunk(data)
	if not (data and data.name) then return true end
	if data.metadata then data.damage = data.metadata end
	for k,v in pairs(blacklist) do
		if v.name == data.name then
			if v.damage then
				if v.damage == data.damage then
					return true
				end
			else
				return true
			end
		end
	end
	return false
end

function isLiquidFuel(data)
	if not (data and data.name) then return true end
	if data.metadata then data.damage = data.metadata end
	for k,v in pairs(liquidfuels) do
		if v.name == data.name then
			if v.damage then
				if v.damage == data.damage then
					return true
				end
			else
				return true
			end
		end
	end
	return false
end

-- detects if the turtle can store the requested amount of a specified block/item
-- if it can, returns true and how much more of specified item could be stored
-- if it cannot, returns false and how many of the specified item would not fit
function canFit(data, amt)
	amt = amt or 1
	if not data.name then
		return false
	end
	for i = 1, 16 do
		local slotData = turtle.getItemDetail(i)
		if (not slotData) or (slotData.name == data.name and slotData.damage == data.damage) then
			amt = amt - turtle.getItemSpace(i)
		end
	end
	if amt <= 0 then
		return true, math.abs(amt)
	else
		return false, amt
	end
end

-- refuels the turtle to a specified target
function refuel(target)
	for i = 1, 16 do
		turtle.select(i)
		if turtle.refuel(0) then
			while turtle.getFuelLevel() < target and turtle.getItemCount(i) > 0 do
				turtle.refuel(1)
			end
		end
		if turtle.getFuelLevel() > target then
			break
		end
	end
	return turtle.getFuelLevel()
end

function tossJunk(direction)
	local direction = direction or "down"
	for i = 1, 16 do
		if turtle.getItemCount(i) > 0 and isJunk(turtle.getItemDetail(i)) then
			turtle.select(i)
			if direction == "up" then
				turtle.dropUp()
			elseif direction == "forward" then
				turtle.drop()
			else
				turtle.dropDown()
			end
		end
	end
end

function findBucket()
	for i = 1, 16 do
		local data = turtle.getItemDetail(i)
		if data and data.name == "minecraft:bucket" then
			turtle.select(i)
			return true
		end
	end
	return false
end

function mineSides()
	turtle.select(1)
	for i = 1, 4 do
		local _, data = turtle.inspect()
		data = b2i(data)
		while turtle.suck(64) do end
		if not isJunk(data) then
			if canFit(data) then
				turtle.dig()
			end
		end
		if isLiquidFuel(data) then
			if findBucket() then turtle.place() turtle.refuel(1) end
		end
		go.right()
	end
end

function mineShaft()
	--local startPos = go.getPos()
	local descended = 0
	while true do
		mineSides()
		local _, data = turtle.inspectDown()
		data = b2i(data)
		if data then
			if isLiquidFuel(data) then
				if findBucket() then turtle.placeDown() turtle.refuel(1) end
			else
				if not turtle.digDown() then break end
			end
		end
		while turtle.attackDown() do end
		if not go.down() then break end
		descended = descended + 1
	end
	return descended
end
