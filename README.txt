LUBU - LUA BUNDLER
==================

LuBu is a Go-based Lua bundler that compiles multiple .lua and .dll modules into a single output file.


HOW IT WORKS
============

1. Reads lubu.json config file
2. Processes all modules (lua/dll)
3. Wraps each module in package.preload for require() support
4. Wraps main file in LUBU_ENTRY_POINT function
5. Outputs single bundled .lua file


CONFIG FILE (lubu.json)
=======================

Required fields:
  - main: string - path to entry point file
  - out: string - path to output bundled file
  - modules: object - module name -> file path mapping

Optional fields:
  - const: object - global constants (string/number/bool only)
  - watcher_delay: number - ms delay for file watcher (0 = disabled)
  - resource: object - runtime file path -> source file path
  - prepare_for_obfuscation: bool - convert numbers to tonumber("N")
  - remove_comments: bool - strip all comments
  - remove_empty_lines: bool - strip empty lines
  - minify: bool - minify output


MODULE SYSTEM
=============

Lua modules:
  - Wrapped in: package['preload']['MODULE_NAME'] = function() ... end
  - Accessed via: require('MODULE_NAME')

DLL modules:
  - Converted to base64
  - Written to temp file at runtime
  - Loaded via package.loadlib


BUNDLED OUTPUT STRUCTURE
========================

1. LuBu header comment
2. DLL write function (if dll modules exist)
3. Obfuscation setup (if enabled)
4. Constants block (LUBU_BUNDLED, LUBU_BUNDLED_AT, user constants)
5. Resources block
6. Module definitions (package.preload)
7. Entry point (LUBU_ENTRY_POINT function call)


GLOBAL VARIABLES IN BUNDLED CODE
================================

LUBU_BUNDLED = true
LUBU_BUNDLED_AT = unix_timestamp
+ any user-defined constants from config


OBFUSCATION DIRECTIVES
======================

---@OBFIGNORE   - start ignore block
---@ENDOBFIGNORE - end ignore block

Code between these markers won't be processed for obfuscation.


USAGE
=====

lubu.exe lubu.json

With watcher_delay > 0, LuBu watches files and rebundles on changes.


EXAMPLE CONFIG
==============

{
    "main": "src/init.lua",
    "out": "dist/bundled.lua",
    "modules": {
        "utils": "src/utils.lua",
        "commands": "src/commands.lua"
    },
    "const": {
        "VERSION": "1.0.0"
    },
    "watcher_delay": 250,
    "remove_comments": true
}


EXAMPLE MODULE (src/utils.lua)
==============================

local Utils = {}

function Utils.log(msg)
    print('[LOG] ' .. msg)
end

return Utils


EXAMPLE MAIN (src/init.lua)
===========================

local Utils = require('utils')
Utils.log('Script loaded!')


PROJECT STRUCTURE
=================

project/
  src/
    init.lua      - entry point
    utils.lua     - utility module
    commands/     - command modules
      test.lua
  dist/
    bundled.lua   - output
  lubu.json       - config
