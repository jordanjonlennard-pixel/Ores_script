local TweenService = game:GetService("TweenService")

----------------------------------------------------------------
-- Remove old ore deposits
----------------------------------------------------------------

local oldFolder = workspace:FindFirstChild("OreDeposits")

if oldFolder then
	oldFolder:Destroy()
end

local oreFolder = Instance.new("Folder")
oreFolder.Name = "OreDeposits"
oreFolder.Parent = workspace

----------------------------------------------------------------
-- Ore settings
----------------------------------------------------------------

local ores = {
	{
		Name = "StoneDeposit",
		MainColor = Color3.fromRGB(125, 130, 145),
		ChunkColor = Color3.fromRGB(190, 195, 205),
		GlowBrightness = 0.8,
		GlowRange = 6,
		Position = Vector3.new(0, 6, 0)
	},

	{
		Name = "CopperDeposit",
		MainColor = Color3.fromRGB(85, 55, 42),
		ChunkColor = Color3.fromRGB(235, 85, 28),
		GlowBrightness = 2,
		GlowRange = 9,
		Position = Vector3.new(13, 6, 0)
	},

	{
		Name = "TinDeposit",
		MainColor = Color3.fromRGB(105, 105, 98),
		ChunkColor = Color3.fromRGB(220, 225, 220),
		GlowBrightness = 1.4,
		GlowRange = 8,
		Position = Vector3.new(26, 6, 0)
	},

	{
		Name = "CoalDeposit",
		MainColor = Color3.fromRGB(18, 20, 27),
		ChunkColor = Color3.fromRGB(42, 48, 58),
		GlowBrightness = 0.7,
		GlowRange = 6,
		Position = Vector3.new(39, 6, 0)
	},

	{
		Name = "IronDeposit",
		MainColor = Color3.fromRGB(72, 82, 98),
		ChunkColor = Color3.fromRGB(155, 165, 180),
		GlowBrightness = 1.2,
		GlowRange = 8,
		Position = Vector3.new(52, 6, 0)
	}
}

----------------------------------------------------------------
-- Create a block
----------------------------------------------------------------

local function createPart(
	name,
	size,
	position,
	color,
	material,
	rotation,
	parent
)
	local part = Instance.new("Part")

	part.Name = name
	part.Shape = Enum.PartType.Block
	part.Size = size
	part.Position = position
	part.Color = color
	part.Material = material
	part.Orientation = rotation
	part.Anchored = true
	part.CanCollide = true

	part.TopSurface = Enum.SurfaceType.Smooth
	part.BottomSurface = Enum.SurfaceType.Smooth

	part.Parent = parent

	return part
end

----------------------------------------------------------------
-- Add glowing light
----------------------------------------------------------------

local function addGlow(part, ore, brightnessMultiplier)
	local light = Instance.new("PointLight")
	light.Name = "OreGlow"
	light.Color = ore.ChunkColor
	light.Brightness = ore.GlowBrightness * brightnessMultiplier
	light.Range = ore.GlowRange * brightnessMultiplier
	light.Shadows = true
	light.Parent = part

	local tween = TweenService:Create(
		light,
		TweenInfo.new(
			1.2,
			Enum.EasingStyle.Sine,
			Enum.EasingDirection.InOut,
			-1,
			true
		),
		{
			Brightness = ore.GlowBrightness * brightnessMultiplier * 1.5,
			Range = ore.GlowRange * brightnessMultiplier + 2
		}
	)

	tween:Play()
end

----------------------------------------------------------------
-- Create a square cupcake-shaped glowing chunk
----------------------------------------------------------------

local function createCupcakeChunk(ore, model, index, chunk)
	local basePosition = ore.Position + chunk.Offset

	local widthVariation = math.random(90, 115) / 100
	local heightVariation = math.random(90, 120) / 100
	local depthVariation = math.random(90, 115) / 100

	-- Wide square bottom
	local baseSize = Vector3.new(
		chunk.Size.X * widthVariation,
		chunk.Size.Y * 0.45 * heightVariation,
		chunk.Size.Z * depthVariation
	)

	-- Smaller square middle
	local middleSize = Vector3.new(
		chunk.Size.X * 0.85 * widthVariation,
		chunk.Size.Y * 0.4 * heightVariation,
		chunk.Size.Z * 0.85 * depthVariation
	)

	-- Raised square top
	local topSize = Vector3.new(
		chunk.Size.X * 0.62 * widthVariation,
		chunk.Size.Y * 0.3 * heightVariation,
		chunk.Size.Z * 0.62 * depthVariation
	)

	local rotation = chunk.Rotation

	local base = createPart(
		"GlowingChunkBase_" .. index,
		baseSize,
		basePosition - Vector3.new(0, chunk.Size.Y * 0.25, 0),
		ore.ChunkColor,
		Enum.Material.Neon,
		rotation,
		model
	)

	local middle = createPart(
		"GlowingChunkMiddle_" .. index,
		middleSize,
		basePosition + Vector3.new(0, chunk.Size.Y * 0.08, 0),
		ore.ChunkColor,
		Enum.Material.Neon,
		Vector3.new(
			rotation.X + math.random(-8, 8),
			rotation.Y + math.random(-10, 10),
			rotation.Z + math.random(-8, 8)
		),
		model
	)

	local top = createPart(
		"GlowingChunkTop_" .. index,
		topSize,
		basePosition + Vector3.new(0, chunk.Size.Y * 0.34, 0),
		ore.ChunkColor,
		Enum.Material.Neon,
		Vector3.new(
			rotation.X + math.random(-12, 12),
			rotation.Y + math.random(-15, 15),
			rotation.Z + math.random(-12, 12)
		),
		model
	)

	addGlow(base, ore, 0.45)
	addGlow(middle, ore, 0.8)
	addGlow(top, ore, 0.5)
end

----------------------------------------------------------------
-- Create one complete cupcake-shaped ore deposit
----------------------------------------------------------------

local function createOreDeposit(ore)
	local model = Instance.new("Model")
	model.Name = ore.Name
	model.Parent = oreFolder

	------------------------------------------------------------
	-- Large square cupcake-style body
	------------------------------------------------------------

	local bodySections = {
		{
			Name = "WideBottom",
			Size = Vector3.new(9.2, 2, 8.2),
			Offset = Vector3.new(0, -2.3, 0),
			Rotation = Vector3.new(0, math.random(-12, 12), 0)
		},

		{
			Name = "LowerBody",
			Size = Vector3.new(9.8, 2.4, 8.8),
			Offset = Vector3.new(0, -0.5, 0),
			Rotation = Vector3.new(
				math.random(-5, 5),
				math.random(-10, 10),
				math.random(-5, 5)
			)
		},

		{
			Name = "MiddleBody",
			Size = Vector3.new(8.8, 2.5, 7.8),
			Offset = Vector3.new(0, 1.5, 0),
			Rotation = Vector3.new(
				math.random(-6, 6),
				math.random(-15, 15),
				math.random(-6, 6)
			)
		},

		{
			Name = "UpperBody",
			Size = Vector3.new(7.6, 2.1, 6.8),
			Offset = Vector3.new(0, 3.5, 0),
			Rotation = Vector3.new(
				math.random(-8, 8),
				math.random(-20, 20),
				math.random(-8, 8)
			)
		},

		{
			Name = "SquareTop",
			Size = Vector3.new(5.8, 1.5, 5.4),
			Offset = Vector3.new(0, 5, 0),
			Rotation = Vector3.new(
				math.random(-10, 10),
				math.random(-25, 25),
				math.random(-10, 10)
			)
		}
	}

	local mainBody

	for index, section in ipairs(bodySections) do
		local bodyPart = createPart(
			section.Name,
			section.Size,
			ore.Position + section.Offset,
			ore.MainColor,
			Enum.Material.Slate,
			section.Rotation,
			model
		)

		if index == 3 then
			mainBody = bodyPart
		end
	end

	------------------------------------------------------------
	-- Additional angular pieces around all sides
	------------------------------------------------------------

	local sideBlocks = {
		-- Left
		{
			Offset = Vector3.new(-4.7, 0.5, -1.6),
			Size = Vector3.new(2, 3.1, 2.9),
			Rotation = Vector3.new(8, 20, -12)
		},

		-- Right
		{
			Offset = Vector3.new(4.7, 0.7, 1.1),
			Size = Vector3.new(2, 3.2, 2.8),
			Rotation = Vector3.new(-5, -25, 12)
		},

		-- Front
		{
			Offset = Vector3.new(-2.8, 2, -4.2),
			Size = Vector3.new(3.1, 2, 1.7),
			Rotation = Vector3.new(10, 15, -8)
		},

		-- Back
		{
			Offset = Vector3.new(2.8, 2.2, 4.1),
			Size = Vector3.new(3.1, 2, 1.7),
			Rotation = Vector3.new(-8, -20, 10)
		},

		-- Lower front
		{
			Offset = Vector3.new(1, -1.8, -3.9),
			Size = Vector3.new(3.6, 1.5, 1.5),
			Rotation = Vector3.new(10, -10, 5)
		},

		-- Lower back
		{
			Offset = Vector3.new(-1, -1.8, 3.9),
			Size = Vector3.new(3.6, 1.5, 1.5),
			Rotation = Vector3.new(-10, 10, -5)
		}
	}

	for index, block in ipairs(sideBlocks) do
		createPart(
			"AngularSideBlock_" .. index,
			block.Size,
			ore.Position + block.Offset,
			ore.MainColor,
			Enum.Material.Slate,
			block.Rotation,
			model
		)
	end

	------------------------------------------------------------
	-- Square cupcake-shaped glowing chunks
	------------------------------------------------------------

	local oreChunks = {
		-- Front side
		{
			Offset = Vector3.new(-2.8, 1.3, -4.65),
			Size = Vector3.new(2.2, 2.4, 1.4),
			Rotation = Vector3.new(10, 20, -8)
		},

		{
			Offset = Vector3.new(0, 2.8, -4.55),
			Size = Vector3.new(2.4, 2.2, 1.4),
			Rotation = Vector3.new(-8, 15, 8)
		},

		{
			Offset = Vector3.new(2.8, 0.5, -4.6),
			Size = Vector3.new(2.1, 2.5, 1.4),
			Rotation = Vector3.new(5, -20, 12)
		},

		-- Back side
		{
			Offset = Vector3.new(-2.6, 1.6, 4.55),
			Size = Vector3.new(2.2, 2.3, 1.4),
			Rotation = Vector3.new(-10, 25, 8)
		},

		{
			Offset = Vector3.new(0.8, 2.8, 4.55),
			Size = Vector3.new(2.4, 2.2, 1.4),
			Rotation = Vector3.new(8, -15, -10)
		},

		{
			Offset = Vector3.new(2.8, -0.3, 4.55),
			Size = Vector3.new(2, 2.2, 1.4),
			Rotation = Vector3.new(12, 20, 5)
		},

		-- Left side
		{
			Offset = Vector3.new(-5.15, 1.4, -1.7),
			Size = Vector3.new(1.4, 2.3, 2.2),
			Rotation = Vector3.new(10, 25, -12)
		},

		{
			Offset = Vector3.new(-5.15, -0.8, 1.6),
			Size = Vector3.new(1.4, 2.2, 2.1),
			Rotation = Vector3.new(-8, -20, 10)
		},

		-- Right side
		{
			Offset = Vector3.new(5.15, 1.6, -1.1),
			Size = Vector3.new(1.4, 2.4, 2.2),
			Rotation = Vector3.new(5, -25, 12)
		},

		{
			Offset = Vector3.new(5.15, -0.6, 1.5),
			Size = Vector3.new(1.4, 2.2, 2.1),
			Rotation = Vector3.new(15, 20, -8)
		},

		-- Top
		{
			Offset = Vector3.new(-2, 5.7, -1.5),
			Size = Vector3.new(2.2, 1.3, 2),
			Rotation = Vector3.new(-15, 20, 8)
		},

		{
			Offset = Vector3.new(1.5, 5.8, 0.8),
			Size = Vector3.new(2.3, 1.3, 2),
			Rotation = Vector3.new(10, -15, -8)
		},

		{
			Offset = Vector3.new(0, 6, -1),
			Size = Vector3.new(1.9, 1.2, 1.8),
			Rotation = Vector3.new(-8, 30, 12)
		},

		-- Bottom
		{
			Offset = Vector3.new(-1.8, -3.35, -1),
			Size = Vector3.new(2.2, 1, 1.9),
			Rotation = Vector3.new(15, -20, 5)
		},

		{
			Offset = Vector3.new(1.5, -3.35, 1),
			Size = Vector3.new(2.3, 1, 1.9),
			Rotation = Vector3.new(-12, 15, -10)
		}
	}

	for index, chunk in ipairs(oreChunks) do
		createCupcakeChunk(ore, model, index, chunk)
	end

	if mainBody then
		model.PrimaryPart = mainBody
	end
end

----------------------------------------------------------------
-- Create all five ores
----------------------------------------------------------------

for _, ore in ipairs(ores) do
	createOreDeposit(ore)
end
