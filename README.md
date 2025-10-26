-- main.lua
-- Protótipo educativo em Love2D (LÖVE)
-- Não use isso para trapacear em jogos de terceiros.
-- Controles:
-- Arrow keys / WASD = mover
-- V = alternar "visual" (mostrar inimigos por cima das paredes)
-- N = alternar noclip (atravessar paredes)
-- + / - = ajustar velocidade

function love.load()
    love.window.setTitle("Protótipo educativo: visual / noclip / speed")
    love.window.setMode(800, 600)

    player = {
        x = 100, y = 100, w = 20, h = 28,
        speed = 140, -- pixels por segundo
        color = {0.2,0.7,1}
    }

    walls = {
        {x=200,y=50,w=20,h=400},
        {x=350,y=150,w=300,h=20},
        {x=600,y=0,w=20,h=300},
        {x=450,y=350,w=200,h=20}
    }

    enemies = {
        {x=500,y=100,r=10, color={1,0.2,0.2}},
        {x=300,y=300,r=10, color={1,0.4,0.2}},
        {x=700,y=450,r=10, color={1,0.6,0.3}},
    }

    ui = {
        showVisual = false, -- mostra inimigos mesmo "atrás" das paredes
        noclip = false
    }

    font = love.graphics.newFont(14)
end

-- AABB collision
local function aabbColl(a, b)
    return a.x < b.x + b.w and b.x < a.x + a.w and
           a.y < b.y + b.h and b.y < a.y + a.h
end

-- Checa se o jogador colide com uma parede
local function collidesWithWall(px, py, pw, ph)
    local pRect = {x=px, y=py, w=pw, h=ph}
    for _, w in ipairs(walls) do
        if aabbColl(pRect, w) then
            return true, w
        end
    end
    return false, nil
end

function love.update(dt)
    -- Input movimento
    local dx, dy = 0,0
    if love.keyboard.isDown("left") or love.keyboard.isDown("a") then dx = dx - 1 end
    if love.keyboard.isDown("right") or love.keyboard.isDown("d") then dx = dx + 1 end
    if love.keyboard.isDown("up") or love.keyboard.isDown("w") then dy = dy - 1 end
    if love.keyboard.isDown("down") or love.keyboard.isDown("s") then dy = dy + 1 end

    -- Normaliza diagonal
    if dx ~= 0 and dy ~= 0 then
        local inv = 1/math.sqrt(2)
        dx = dx * inv; dy = dy * inv
    end

    local nextX = player.x + dx * player.speed * dt
    local nextY = player.y + dy * player.speed * dt

    if ui.noclip then
        -- atravessa paredes (apenas no protótipo local)
        player.x = nextX
        player.y = nextY
    else
        -- teste simples de colisão: tentar mover em X e Y separadamente
        if not collidesWithWall(nextX, player.y, player.w, player.h) then
            player.x = nextX
        end
        if not collidesWithWall(player.x, nextY, player.w, player.h) then
            player.y = nextY
        end
    end

    -- (opcional) lógica de inimigos simples: pulsam para ficar visíveis
    for i,e in ipairs(enemies) do
        e.pulse = (math.sin(love.timer.getTime()*2 + i) + 1) * 0.5
    end
end

function love.draw()
    love.graphics.clear(0.06,0.08,0.12)

    -- Desenha paredes
    love.graphics.setColor(0.15,0.16,0.18)
    for _, w in ipairs(walls) do
        love.graphics.rectangle("fill", w.x, w.y, w.w, w.h)
    end

    -- Se o modo visual estiver ligado, desenhar inimigos por cima das paredes
    if ui.showVisual then
        -- desenha inimigos primeiro com efeito de "glow" por cima
        for _, e in ipairs(enemies) do
            local alpha = 0.6 + 0.4 * e.pulse
            love.graphics.setColor(e.color[1], e.color[2], e.color[3], alpha)
            love.graphics.circle("fill", e.x, e.y, e.r + 6 * e.pulse)
            love.graphics.setColor(e.color[1], e.color[2], e.color[3], 1)
            love.graphics.circle("fill", e.x, e.y, e.r)
        end
    else
        -- modo padrão: desenha inimigos, mas some se atrás da parede (simples teste de visibilidade)
        for _, e in ipairs(enemies) do
            local visible = true
            -- método simplificado: verifica se a posição do inimigo está dentro de alguma parede
            for _, w in ipairs(walls) do
                if e.x > w.x and e.x < w.x + w.w and e.y > w.y and e.y < w.y + w.h then
                    visible = false
                    break
                end
            end
            if visible then
                love.graphics.setColor(e.color)
                love.graphics.circle("fill", e.x, e.y, e.r)
            end
        end
    end

    -- Desenha jogador
    love.graphics.setColor(player.color)
    love.graphics.rectangle("fill", player.x, player.y, player.w, player.h)

    -- Desenha UI texto
    love.graphics.setFont(font)
    love.graphics.setColor(1,1,1)
    love.graphics.print(("Velocidade: %.1f px/s"):format(player.speed), 10, 10)
    love.graphics.print(("Visual (V): %s"):format(ui.showVisual and "ON" or "OFF"), 10, 30)
    love.graphics.print(("Noclip (N): %s"):format(ui.noclip and "ON" or "OFF"), 10, 50)
    love.graphics.print(" + / - para ajustar velocidade", 10, 70)

    -- ajuda rápida
    love.graphics.setColor(1,1,1,0.18)
    love.graphics.printf("Protótipo educativo — não use para trapacear em jogos de terceiros", 10, love.graphics.getHeight()-40, love.graphics.getWidth()-20, "left")
end

function love.keypressed(key)
    if key == "v" then
        ui.showVisual = not ui.showVisual
    elseif key == "n" then
        ui.noclip = not ui.noclip
    elseif key == "kp+" or key == "=" or key == "+" then
        player.speed = player.speed + 20
    elseif key == "kp-" or key == "-" then
        player.speed = math.max(20, player.speed - 20)
    elseif key == "escape" then
        love.event.quit()
    end
end
