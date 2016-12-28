--[[
tfs version 0.1.0
 
The MIT License (MIT)
Copyright (c) 2016 CrazedProgrammer
 
Permission is hereby granted, free of charge, to any person obtaining a copy of this software and
associated documentation files (the "Software"), to deal in the Software without restriction,
including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense,
and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do
so, subject to the following conditions:
 
The above copyright notice and this permission notice shall be included in all copies or
substantial portions of the Software.
 
THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED,
INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A
PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR
COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN
AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION
WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
]]

local _fs = { }
for k, v in pairs(_G.fs) do
	_fs[k] = v
end
for k, _ in pairs(_fs) do
	_G.fs[k] = nil
end

local fsTree = { }

local function getPath(path)
	if path == "" or path == "/" then return "" end
	if _fs.getDir(path) == "" then
		return _fs.getName(path)
	else
		return _fs.getDir(path).."/".._fs.getName(path)
	end
end

local function getParts(path)
	path = getPath(path)
	if path == "" then return { } end
	local parts, index = { }, 1
	for i = 1, #path do
		if path:sub(i, i) == "/" then
			parts[#parts + 1] = path:sub(index, i - 1)
			index = i + 1
		end
	end
	parts[#parts + 1] = path:sub(index, #path)
	return parts
end

local function access(path)
	local parts = getParts(path)
	if parts[1] ~= "rom" then
		local current = fsTree
		for i = 1, #parts do
			if type(current) == "table" then
				if current[parts[i]] then
					current = current[parts[i]]
				else
					return nil
				end
			else
				return nil
			end
		end
		if type(current) == "table" then
			local t = { }
			for k, _ in pairs(current) do
				t[#t + 1] = k
			end
			if #parts == 0 then
				t[#t + 1] = "rom"
			end
			return t
		else
			return current
		end
	else
		if _fs.exists(path) then
			if _fs.isDir(path) then
				return _fs.list(path)
			else
				local handle = _fs.open(path, "r")
				local data = handle.readAll()
				handle.close()
				return data
			end
		else
			return nil
		end
	end
end

function fs.list(path)
	local dir = access(path)
	if type(dir) == "table" then
		return dir
	else
		error("Not a directory")
	end
end

function fs.exists(path)
	return access(path) ~= nil
end

function fs.isDir(path)
	return type(access(path)) == "table"
end

function fs.isReadOnly(path)
	return getParts(path)[1] == "rom"
end

fs.getName = _fs.getName

function fs.getDrive(path)
	if not access(path) then return nil end
	return (getParts(path)[1] == "rom") and "rom" or "hdd"
end

function fs.getSize(path)
	local file = access(path)
	if type(file) == "string" then
		return #file
	elseif file == nil then
		error("No such file")
	else
		return 0
	end
end

function fs.getFreeSpace(path)
	return (getParts(path)[1] == "rom") and 0 or 99999999
end

function fs.makeDir(path)
	local parts = getParts(path)
	if parts[1] ~= "rom" then
		local current = fsTree
		for i = 1, #parts do
			if type(current[parts[i]]) ~= "string" then
				if type(current[parts[i]]) == "nil" then
					current[parts[i]] = { }
				end
				current = current[parts[i]]
			else
				error("File exists")
			end
		end
	else
		error("Access Denied") -- great consistency you've got there, Dan
		-- really, this is pleasing my OCD
	end
end

function fs.delete(path)
	local parts = getParts(path)
	if parts[1] ~= "rom" then
		local current = fsTree
		for i = 1, #parts do
			if i == #parts then
				current[parts[i]] = nil
			elseif type(current[parts[i]]) == "table" then
				current = current[parts[i]]
			else
				return
			end
		end
	else
		error("Access Denied")
	end
end

function fs.move(frompath, topath)
	local data = access(frompath)
	if type(data) == "table" then
		fs.makeDir(topath)
		fs.delete(frompath)
	elseif type(data) == "string" then
		local h = fs.open(topath, "wb")
		if h then
			for i = 1, #data do
				h.write(string.byte(data, i))
			end
			h.close()
		else
			error("File exists")
		end
		fs.delete(frompath)
	else
		error("No matching files")
	end
end

function fs.copy(frompath, topath)
	local data = access(frompath)
	if type(data) == "table" then
		fs.makeDir(topath)
	elseif type(data) == "string" then
		local h = fs.open(topath, "wb")
		if h then
			for i = 1, #data do
				h.write(string.byte(data, i))
			end
			h.close()
		else
			error("File exists")
		end
	else
		error("No matching files")
	end
end

fs.combine = _fs.combine

function fs.open(path, mode)
	local parts = getParts(path)
	if mode == "r" or mode == "rb" then
		local data = access(path)
		if type(data) ~= "string" then return end
		
		local handle = { }
		local closed = false
		local index = 1
		if mode == "rb" then
			function handle.read()
				if closed then error("Stream closed") end
				if index > #data then return end
				local byte = string.byte(data, index)
				index = index + 1
				return byte
			end
		else
			function handle.readLine()
				if closed then error("Stream closed") end
				if index > #data then return nil end
				for i = index, #data do
					if data:sub(i, i) == "\n" then
						local str = data:sub(index, i - 1)
						index = i + 1
						return str
					end
				end
				local str = data:sub(index, #data)
				index = #data + 1
				return str
			end
			function handle.readAll()
				if closed then error("Stream closed") end
				if index > #data then return "" end
				local str = data:sub(index, #data - ((data:sub(#data, #data) == "\n") and 1 or 0))
				index = #data + 1
				return str
			end
		end
		function handle.close()
			closed = true
		end
		return handle
	elseif mode == "w" or mode == "wb" or mode == "a" or mode == "ab" then
		if parts[1] == "rom" then
			error("Access Denied")
		end
		if not access(path) then
			if #parts > 1 then
				local dir = ""
				for i = 1, #parts - 1 do
					dir = dir.."/"
				end
				if not pcall(fs.makeDir, dir) then return end
			end
		elseif type(access(path)) == "table" then return end

		local current = fsTree
		for i = 1, #parts - 1 do
			if type(current[parts[i]]) ~= "table" then return end
			current = current[parts[i]]
		end
		local file = parts[#parts]

		local data = {""}
		if current[file] then
			if mode:sub(1, 1) == "w" then
				current[file] = ""
			else
				data = current[file]
			end
		else
			current[file] = ""
		end

		local handle = { }
		local closed = false
		if mode == "wb" or mode == "ab" then
			function handle.write(byte)
				if closed then error("Stream closed") end
				data[#data + 1] = string.char(byte)
			end
		else
			function handle.write(str)
				if closed then error("Stream closed") end
				data[#data + 1] = str
			end
			function handle.writeLine(str)
				if closed then error("Stream closed") end
				data[#data + 1] = str.."\n"
			end
		end
		function handle.flush()
			current[file] = table.concat(data)
			data = {current[file]}
		end
		function handle.close()
			handle.flush()
			closed = true
		end
		return handle
	else
		error("Unsupported mode")
	end
end

function fs.find(wildcard)
	wildcard = getPath(wildcard)
	local paths = { }
	
	local function addPaths(path, dir)
		for i = 1, #dir do
			local p = path.."/"..dir[i]
			local c = access(p)
			paths[#paths + 1] = p:sub(2)
			if type(c) == "table" then
				addPaths(p, c)
			end
		end
	end
	addPaths("", access(""))

	local _, wslashes = wildcard:gsub("%/", "")
	local pattern = wildcard:gsub("%*", "[^/]+")
	local results = { }
	for i = 1, #paths do
		if paths[i]:match(pattern) then
			local __, pslashes = paths[i]:gsub("%/", "")
			if wslashes == pslashes then
				results[#results + 1] = paths[i]
			end
		end
	end
	return results
end

fs.getDir = _fs.getDir

function fs.complete(partialname, path, includefiles, includeslashes) -- todo
	local dir = access(path)
	if type(dir) ~= "table" then return { } end
	local completes = { }
	for i = 1, #dir do
		if dir[i]:sub(1, #partialname) == partialname then
			if type(access(path.."/"..dir[i])) ~= "string" or includefiles then
				if includeslashes then
					completes[#completes + 1] = dir[i]:sub(#partialname + 1).."/"
				end
				completes[#completes + 1] = dir[i]:sub(#partialname + 1)
			end
		end
	end
	return completes
end

if ({...})[1] == "-i" then
	local function scan(path)
		local items = _fs.list(path)
		for i = 1, #items do
			if not (path == "" and items[i] == "rom") then
				local p = path.."/"..items[i]
				if _fs.isDir(p) then
					fs.makeDir(p)
					scan(p)
				else
					local ih = _fs.open(p, "rb")
					if ih then
						local h = fs.open(p, "wb")
						local byte = ih.read()
						while byte do
							h.write(byte)
							byte = ih.read()
						end
						ih.close()
						h.close()
					end
				end
			end
		end
	end
	scan("")
end