-- Basic Shelly Dimmer Plugin
-- Raymond Dean @ VFC Cran
-- Apri 2026

PluginInfo = {
    Name = "Lighting~SHELLY_DIMMER",
    Version = "0.1.3A",
    Id = "ShellyDimmerProd_0.1A@1447",
    Author = "Raymond Dean @ VFC Cran",
    Description = "Basic Shelly Dimmer Plugin",
    ShowDebug = true,
}
-- ============================================================
--  Pretty name
-- ============================================================
function GetPrettyName(props)
  return string.format(
    "Shelly Dimmer %s [%s]",
    props["Device Generation"].Value,
    props["IP Address"].Value
  )
end

-- ============================================================
--  Properties
-- ============================================================
function GetProperties()
  return {
    {
      Name  = "IP Address",
      Type  = "string",
      Value = "0.0.0.0"
    },
    {
      Name    = "Device Generation",
      Type    = "enum",
      Choices = {"Gen1", "Gen2"},
      Value   = "Gen1"
    },
    {
      Name  = "Poll Interval (s)",
      Type  = "integer",
      Min   = 1,
      Max   = 60,
      Value = 5
    }
  }
end

function RectifyProperties(props)
  return props
end

-- ============================================================
--  Controls
-- ============================================================
function GetControls(props)
  return {
    -- Status
    {
      Name         = "online",
      ControlType  = "Indicator",
      DefaultValue = "false",
      UserPin      = true,
      PinStyle     = "Output",
      Count        = 1
    },
    {
      Name          = "status_message",
      ControlType   = "Indicator",
      IndicatorType = "Text",
      DefaultValue  = "",
      UserPin       = true,
      PinStyle      = "Output",
      Count         = 1
    },
    -- Power toggle
    {
      Name         = "power",
      ControlType  = "Button",
      ButtonType   = "Toggle",
      DefaultValue = "0",
      UserPin      = true,
      PinStyle     = "Both",
      Count        = 1
    },
    -- Brightness 0–100 %
    {
      Name         = "brightness",
      ControlType  = "Knob",
      ControlUnit  = "Percent",
      DefaultValue = "100",
      Min          = 0,
      Max          = 100,
      UserPin      = true,
      PinStyle     = "Both",
      Count        = 1
    },
    -- Transition time (ms) – converted to seconds for Gen2; minimum 1000
    {
      Name         = "transition",
      ControlType  = "Knob",
      ControlUnit  = "Integer",
      DefaultValue = "1000",
      Min          = 1000,
      Max          = 5000,
      UserPin      = true,
      PinStyle     = "Both",
      Count        = 1
    }
  }
end

-- ============================================================
--  Layout
-- ============================================================
function GetControlLayout(props)
  local layout   = {}
  local graphics = {}

  local W  = 220
  local gen = props["Device Generation"].Value

  -- Header
  table.insert(graphics, {
    Type       = "Header",
    Text       = "Shelly Dimmer (" .. gen .. ")",
    HTextAlign = "Center",
    Color      = {40, 40, 40},
    FontSize   = 13,
    Position   = {0, 0},
    Size       = {W, 32}
  })

  -- Online LED
  layout["online"] = {
    PrettyName     = "Online",
    Style          = "Indicator",
    Color          = {0, 200, 0},
    OffColor       = {200, 0, 0},
    UnlinkOffColor = true,
    Position       = {W - 28, 6},
    Size           = {20, 20}
  }

  -- Status text
  layout["status_message"] = {
    PrettyName = "Status",
    Style      = "Text",
    Position   = {8, 36},
    Size       = {W - 16, 18}
  }

  -- Power button
  layout["power"] = {
    PrettyName     = "Power",
    Style          = "Button",
    ButtonStyle    = "Toggle",
    Legend         = "Power",
    Color          = {0, 180, 0},
    OffColor       = {180, 0, 0},
    UnlinkOffColor = true,
    Position       = {8, 62},
    Size           = {90, 36}
  }

  -- Brightness knob
  layout["brightness"] = {
    PrettyName = "Brightness",
    Style      = "Knob",
    Position   = {110, 62},
    Size       = {100, 100}
  }

  -- Transition knob
  layout["transition"] = {
    PrettyName = "Transition (ms)",
    Style      = "Knob",
    Position   = {8, 106},
    Size       = {90, 90}
  }

  return layout, graphics
end

-- ============================================================
--  Runtime
-- ============================================================
if Controls then

  local ip           = Properties["IP Address"].Value
  local gen          = Properties["Device Generation"].Value   -- "Gen1" | "Gen2"
  local pollInterval = Properties["Poll Interval (s)"].Value
  local isGen2       = (gen == "Gen2")
  local updating     = false   -- true while poll is writing controls; blocks EventHandlers

  -- ── Shared helpers ────────────────────────────────────────

  local function setStatus(online, msg)
    Controls.online.Boolean        = online
    Controls.status_message.String = msg or ""
  end

  -- Lightweight JSON parser for flat Shelly response objects.
  -- Handles booleans, numbers, and strings — no library required.
  local function parseJson(s)
    local t = {}
    for k, v in s:gmatch('"([%w_]+)"%s*:%s*(true)') do
      t[k] = true
    end
    for k, v in s:gmatch('"([%w_]+)"%s*:%s*(false)') do
      t[k] = false
    end
    for k, v in s:gmatch('"([%w_]+)"%s*:%s*(-?%d+%.?%d*)') do
      if t[k] == nil then t[k] = tonumber(v) end
    end
    for k, v in s:gmatch('"([%w_]+)"%s*:%s*"([^"]*)"') do
      if t[k] == nil then t[k] = v end
    end
    return t
  end

  -- ── Gen1 helpers ──────────────────────────────────────────

  --  GET http://<ip>/light/0?turn=on&brightness=75&transition=500
  local function gen1Url(params)
    local base = string.format("http://%s/light/0", ip)
    if params and #params > 0 then
      return base .. "?" .. table.concat(params, "&")
    end
    return base
  end

  local function gen1SendCommand(on, brightness, transitionMs)
    HttpClient.Download({
      Url     = gen1Url({
        "turn="       .. (on and "on" or "off"),
        "brightness=" .. math.floor(brightness),
        "transition=" .. math.floor(transitionMs)
      }),
      Timeout = 5,
      EventHandler = function(tbl, code, data, err)
        if err then
          setStatus(false, "Error: " .. tostring(err))
        elseif code ~= 200 then
          setStatus(false, "HTTP " .. tostring(code))
        end
      end
    })
  end

  local function gen1Poll()
    HttpClient.Download({
      Url     = gen1Url(nil),
      Timeout = 5,
      EventHandler = function(tbl, code, data, err)
        if err or code ~= 200 then
          setStatus(false, "Offline")
          return
        end
        local p = parseJson(data)
        if not p then
          setStatus(false, "Parse error")
          return
        end
        -- Gen1 fields: ison, brightness
        updating = true
        Controls.power.Boolean    = p.ison       or false
        Controls.brightness.Value = p.brightness or 0
        updating = false
        setStatus(true, string.format(
          "Online | %s | %d%%",
          p.ison and "ON" or "OFF",
          p.brightness or 0
        ))
      end
    })
  end

  -- ── Gen2 helpers ──────────────────────────────────────────

  --  POST http://<ip>/rpc/Light.Set
  --  Body: { "id": 0, "on": true, "brightness": 75, "transition_duration": 0.5 }
  local function gen2SendCommand(on, brightness, transitionMs)
    -- Gen2 supports GET with query params — avoids POST/body issues in Q-SYS
    local url = string.format(
      "http://%s/rpc/Light.Set?id=0&on=%s&brightness=%d&transition_duration=%.3f",
      ip,
      on and "true" or "false",
      math.floor(brightness),
      transitionMs / 1000.0
    )
    print("[ShellyDimmer] GET " .. url)
    HttpClient.Download({
      Url     = url,
      Timeout = 5,
      EventHandler = function(tbl, code, data, err)
        if err then
          print("[ShellyDimmer] CMD error: " .. tostring(err))
          setStatus(false, "Command error")
        elseif code ~= 200 then
          print("[ShellyDimmer] CMD HTTP " .. tostring(code) .. " | " .. tostring(data))
          setStatus(false, "HTTP " .. tostring(code))
        else
          print("[ShellyDimmer] CMD OK: " .. tostring(data))
        end
      end
    })
  end

  --  POST http://<ip>/rpc/Light.GetStatus   Body: { "id": 0 }
  local function gen2Poll()
    HttpClient.Download({
      Url     = string.format("http://%s/rpc/Light.GetStatus?id=0", ip),
      Timeout = 5,
      EventHandler = function(tbl, code, data, err)
        if err or code ~= 200 then
          setStatus(false, "Offline")
          return
        end
        local p = parseJson(data)
        if not p then
          setStatus(false, "Parse error")
          return
        end
        -- Gen2 fields: output (bool), brightness (number)
        updating = true
        Controls.power.Boolean    = p.output     or false
        Controls.brightness.Value = p.brightness or 0
        updating = false
        setStatus(true, string.format(
          "Online | %s | %d%%",
          p.output and "ON" or "OFF",
          p.brightness or 0
        ))
      end
    })
  end

  -- ── Dispatch to correct generation ────────────────────────

  local function sendCommand(on, brightness, transitionMs)
    local t = (transitionMs == 0) and 1000 or transitionMs
    if isGen2 then
      gen2SendCommand(on, brightness, t)
    else
      gen1SendCommand(on, brightness, t)
    end
  end

  local function pollDevice()
    if isGen2 then
      gen2Poll()
    else
      gen1Poll()
    end
  end

  -- ── Control event handlers ────────────────────────────────

  Controls.power.EventHandler = function()
    print("[ShellyDimmer] power EventHandler fired, updating=" .. tostring(updating))
    if updating then return end
    print("[ShellyDimmer] Power → " .. tostring(Controls.power.Boolean))
    sendCommand(
      Controls.power.Boolean,
      Controls.brightness.Value,
      Controls.transition.Value
    )
  end

  Controls.brightness.EventHandler = function()
    print("[ShellyDimmer] brightness EventHandler fired, updating=" .. tostring(updating))
    if updating then return end
    print("[ShellyDimmer] Brightness → " .. tostring(Controls.brightness.Value))
    sendCommand(Controls.power.Boolean, Controls.brightness.Value, Controls.transition.Value)
  end

  -- ── Poll timer ────────────────────────────────────────────

  local pollTimer = Timer.New()
  pollTimer.EventHandler = pollDevice
  pollTimer:Start(pollInterval)

  -- Initial fetch
  pollDevice()

  print("[ShellyDimmer] Initialized — gen=" .. gen .. " ip=" .. ip .. " poll=" .. tostring(pollInterval) .. "s")

end  -- if Controls
