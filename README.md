# refine-unconsious
 A knockout script if you get hit by unarmed/weapons you will get unconsious 

## Need Unique Script Experience?
- [Tebex](https://refined.tebex.io/)

## Need Assistance?
- [Discord](https://discord.gg/Va9YArM6uW)

## Dependency
- [refine-radialbar](https://github.com/HadeJude/refine-radialbar)

### Features
 * Entity Playing even tho you get hit
 * Configurable weapons
 * Optimized Using Refine-radialbar <3
 * Sync and progress can't be cancel 

---

- [Demos](#demos)
- [Installation](#installation)
---

## Demos
- [Video](https://youtu.be/J644eoAdeSk)

---

## Installation
* Look for `qb-ambulancejob` and open `client` folder
* Open `dead.lua` and search `gameEventTriggered` 
* Now Check the line and check the example given.


Old `qb-ambulancejob` Function:
```lua
    AddEventHandler('gameEventTriggered', function(event, data)
        if event == "CEventNetworkEntityDamage" then
            local victim, attacker, victimDied, weapon = data[1], data[2], data[4], data[7]
            if not IsEntityAPed(victim) then return end
            if victimDied and NetworkGetPlayerIndexFromPed(victim) == PlayerId() and IsEntityDead(PlayerPedId()) then
                if not InLaststand then
                    SetLaststand(true)  
                elseif InLaststand and not isDead then
                    SetLaststand(false)
                    local playerid = NetworkGetPlayerIndexFromPed(victim)
                    local playerName = GetPlayerName(playerid) .. " " .. "("..GetPlayerServerId(playerid)..")" or Lang:t('info.self_death')
                    local killerId = NetworkGetPlayerIndexFromPed(attacker)
                    local killerName = GetPlayerName(killerId) .. " " .. "("..GetPlayerServerId(killerId)..")" or Lang:t('info.self_death')
                    local weaponLabel = QBCore.Shared.Weapons[weapon].label or 'Unknown'
                    local weaponName = QBCore.Shared.Weapons[weapon].name or 'Unknown'
                    TriggerServerEvent("qb-log:server:CreateLog", "death", Lang:t('logs.death_log_title', {playername = playerName, playerid = GetPlayerServerId(playerid)}), "red", Lang:t('logs.death_log_message', {killername = killerName, playername = playerName, weaponlabel = weaponLabel, weaponname = weaponName}))
                    deathTime = Config.DeathTime
                    OnDeath()
                    DeathTimer()
                end
            end
        end
    end)
```
New `qb-ambulancejob` Function:
```lua
AddEventHandler('gameEventTriggered', function(event, data)
    if event == "CEventNetworkEntityDamage" then
        local victim, attacker, victimDied, weapon = data[1], data[2], data[4], data[7]
        if not IsEntityAPed(victim) then return end
        if victimDied and NetworkGetPlayerIndexFromPed(victim) == PlayerId() and IsEntityDead(PlayerPedId()) then
            if not InLaststand then
                local isMelee = IsWeaponMelee(weapon) -- Check if the weapon used is melee
                if HasPedBeenDamagedByWeapon(PlayerPedId(), GetHashKey('WEAPON_UNARMED'), 0) or isMelee then
                    local ped = PlayerPedId()
                    local pos = GetEntityCoords(ped)
                    NetworkResurrectLocalPlayer(pos.x, pos.y, pos.z, GetEntityHeading(ped), true, false)
                    exports['refine-radialbar']:Custom({
                        Async = true,
                        canCancel = false,
                        deadCancel = true,  
                        Duration = 19000, 
                        Color = "rgba(224, 53, 40, 1.0)",  
                        Label = "Unconscious",
                        Animation = {
                            animDict = "missarmenian2", 
                            anim = "drunk_loop",
                            flags = 1,
                        },
                        DisableControls = {
                            disableMovement = true,
                            disableCarMovement = true,
                            disableMouse = false,
                            disableCombat = true,
                        },    
                        onStart = function()
                            KnockOut = true
                            CreateThread(function()
                                while KnockOut do
                                    local sleep = 1000
                                    if KnockOut then
                                        sleep = 5
                                        if not IsEntityPlayingAnim(ped, "missarmenian2", "drunk_loop", 3) then
                                            loadAnimDict("missarmenian2")
                                            TaskPlayAnim(ped, "missarmenian2", "drunk_loop", 1.0, 1.0, -1, 1, 0, 0, 0, 0)
                                        end
                                    else
                                        break
                                    end
                                    Wait(sleep)
                                end
                            end)
                        end,
                        onComplete = function(cancelled)
                            if cancelled then
                                KnockOut = false
                            else
                                KnockOut = false
                                ClearPedTasks(ped)
                                SetEntityHealth(ped, math.random(100, 175))
                            end
                        end
                    })
                else
                    SetLaststand(true)
                end
            elseif InLaststand and not isDead then
                SetLaststand(false)
                local playerid = NetworkGetPlayerIndexFromPed(victim)
                local playerName = GetPlayerName(playerid) .. " " .. "("..GetPlayerServerId(playerid)..")" or Lang:t('info.self_death')
                local killerId = NetworkGetPlayerIndexFromPed(attacker)
                local killerName = GetPlayerName(killerId) .. " " .. "("..GetPlayerServerId(killerId)..")" or Lang:t('info.self_death')
                local weaponLabel = QBCore.Shared.Weapons[weapon].label or 'Unknown'
                local weaponName = QBCore.Shared.Weapons[weapon].name or 'Unknown'
                TriggerServerEvent("qb-log:server:CreateLog", "death", Lang:t('logs.death_log_title', {playername = playerName, playerid = GetPlayerServerId(playerid)}), "red", Lang:t('logs.death_log_message', {killername = killerName, playername = playerName, weaponlabel = weaponLabel, weaponname = weaponName}))
                deathTime = Config.DeathTime
                OnDeath()
                DeathTimer()
            end
        end
    end
end)

-- Function to check if a weapon is melee
local meleeWeaponNames = {
    "WEAPON_FLASHLIGHT",
    "WEAPON_KNUCKLE",
    "WEAPON_NIGHTSTICK",
    "WEAPON_POOLCUE",
    "WEAPON_WRENCH",
    "WEAPON_HAMMER",
    "WEAPON_GOLFCLUB",
    "WEAPON_CROWBAR",
    "WEAPON_BAT",
    -- Add more melee weapon names here as needed
}

-- Function to check if a weapon is melee
function IsWeaponMelee(weaponHash)
    -- Check if the weapon hash is in the list of melee weapon hashes
    for _, name in ipairs(meleeWeaponNames) do
        if weaponHash == GetHashKey(name) then
            return true
        end
    end
    return false
end
```
---
