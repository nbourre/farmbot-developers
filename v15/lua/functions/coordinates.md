---
title: "Coordinates"
slug: "coordinates"
description: "List of coordinate Lua functions in FarmBot OS"
---

# coordinate(x, y, z)

**Generates a coordinate** for use in location-based functions such as `move_absolute` and `check_position`.

```lua
coordinate(1.0, 20, 30)
-- Returns {x = 1.0, y = 20,  z = 30}
```

# check_position(coordinate, tolerance)

Returns `true` if the device is **within the tolerance range** of the provided coordinate.

```lua
if check_position({x = 0, y = 0,  z = 0}, 1.23) then
  toast("FarmBot is at the home position", "success")
else
  toast("FarmBot is not at the home position", "warn")
end
```

```lua
home = coordinate(0, 0, 0)
if check_position(home, 0.5) then
  toast("FarmBot is at the home position", "success")
else
  toast("FarmBot is not at the home position", "warn")
end
```

# garden_size()

Returns a table with an `x`, `y`, and `z` attribute representing the **length, width, and height in mm of FarmBot's working volume** according to the **AXIS LENGTH** settings.

```lua
size = garden_size()
toast("Length: " .. size.x .. "mm")
toast("Width: " .. size.y .. "mm")
toast("Height: " .. size.z .. "mm")
```

# get_seed_tray_cell(tray, cell)

Calculates the **coordinates of a seed tray cell**, such as `B3`, based on the cell label and the coordinates of the center of the seed tray. See the [Pick from Seed Tray featured sequence](https://my.farm.bot/app/shared/sequence/32) for an example.

```lua
tray = variable("Seed Tray")
cell_label = variable("Seed Tray Cell")
cell = get_seed_tray_cell(tray, cell_label)
cell_depth = 5

-- Send message with cell info
local cell_coordinates = " (" .. cell.x .. ", " .. cell.y .. ", " .. cell.z - cell_depth .. ")"
toast("Picking up seed from cell " .. cell_label .. cell_coordinates)

-- Safe Z move to above the cell
move_absolute({
    x = cell.x,
    y = cell.y,
    z = cell.z + 25,
    safe_z = true
})
```

# get_xyz()

Gets the current **x, y, and z coordinates** of the FarmBot.

```lua
position = get_xyz()
toast("FarmBot's X coordinate is: " .. position.x)
```

# grid(params)

**Generates a grid of coordinates** based on the provided parameters: `grid_points`, `spacing`, `start`, and `offset`, where each parameter is a table with `x`, `y`, and `z` attributes. Returns an iterator function that is called once per cell (see example below).

|Parameter                 |Description|
|--------------------------|-----------|
|`grid_points`             |The number of grid points along each axis. Must be at least 1 for x, y, and z.
|`spacing`                 |The distance in millimeters between grid points along each axis.
|`start` (optional)        |The coordinates for the first point in the grid. Defaults to (0,0,0).
|`offset` (optional)       |Applies an offset to each grid point. Useful when the end effector of FarmBot (eg: the camera lens) is offset from the center of the tool head. Defaults to 0 for each axis.

{%
include callout.html
type="info"
content="Another optional argument, `ignore_bounds`, may be set to `true` to allow the generation of grid points outside of FarmBot's working volume. Otherwise, `grid()` will only return points that are within the working volume as determined by the **AXIS LENGTH** settings."
%}

```lua
-- Generate a grid of 3x2x1 points, with a spacing of 100mm between each point
local grid_points = {x = 3, y = 2, z = 1}
local spacing = {x = 100, y = 100, z = 0}
local grid = grid({
    grid_points = grid_points,
    spacing = spacing
})

-- Move to each point in the grid
grid.each(function(cell)
  toast("Moving to cell " .. cell.count .. ": (" .. cell.x .. ", " .. cell.y .. ", " .. cell.z .. ")")
  move({
    x = cell.x,
    y = cell.y,
    z = cell.z
  })
end)
```

{%
include callout.html
type="info"
content="The anonymous function passed to `grid.each()` runs in a separate thread. This means you cannot directly access variables from the main thread inside the function. If you need to use external values, consider passing them through the `cell` object."
%}

# group(id)

Returns a table of **current group member IDs**, sorted by the group's **SORT BY** method.

```lua
group_members = group(1234)
for i,member in ipairs(group_members) do
    plant = api({
        method = "get",
        url = "/api/points/" .. member
    })
    move_absolute(plant.x, plant.y, 0)
end
```

{%
include callout.html
type="info"
content="Find a group's ID by navigating to the group in the web app and copying the number at the end of the URL."
%}

# safe_z()

Returns the value of the **[SAFE HEIGHT](https://my.farm.bot/app/designer/settings?highlight=safe_height)** setting.

```lua
-- Display the current Safe Height
toast("Safe Z Height: " .. safe_z())

-- Move FarmBot's Z-axis to the Safe Height
move{z=safe_z()}
```

{%
include callout.html
type="info"
content="`safe_z()` on it's own does not initiate a movement, and it should not be confused with adding a `safe_z=true` argument to a `move` or `move_absolute` command."
%}


# soil_height(x, y)

Given an X and Y coordinate, returns a best-effort estimate of the **Z axis height of the soil**.

```lua
x = 100
y = 300
soil_height = soil_height(x, y)
toast("Distance to soil at (" .. x .. ", " .. y .. "): " .. soil_height)
```

{%
include callout.html
type="info"
content="This function requires at least 3 soil height measurements. When there are less than 3 measurements available, it will return the **SOIL HEIGHT** setting from the device settings page."
%}

# sort(points, method)

**Sorts** the given table of points using the chosen **sorting method**.

```lua
points = group(1234)
sorted_points = sort(points, "xy_alternating")
toast("Second point ID is: " .. sorted_points[2])
```

The following sorting methods are available. See [point group sorting](../../other/how-it-works/point-group-sorting.md) for additional details.

- `xy_ascending`
- `yx_ascending`
- `xy_descending`
- `yx_descending`
- `xy_alternating`
- `yx_alternating`
- `nn` (Nearest Neighbor)
- `random`