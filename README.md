import pygame, random, math
pygame.init()
W,H=800,600
tela=pygame.display.set_mode((W,H))
pygame.display.set_caption("Labirinto Matemático")

azul,vermelho,amarelo,verde,branco,preto=(0,120,255),(255,0,0),(255,255,0),(0,255,0),(255,255,255),(0,0,0)
player=pygame.Rect(50,50,25,25)
porta=pygame.Rect(740,540,40,40)
vel,nivel,pontos,fim,vida=5,1,0,False,3
entrada,resposta,pergunta="",0,None
inimigos,blocos,paredes,particulas=[],[],[],[]
t_cor=0
clock=pygame.time.Clock()
contas_restantes=0

def gerar_labirinto():
    global paredes, inimigos, blocos, player, porta, entrada, pergunta, contas_restantes
    paredes=[]; blocos=[]; inimigos=[]; player.topleft=(50,50); entrada=""; pergunta=None
    porta.topleft=(740,540)
    for _ in range(15+nivel*3):
        x=random.randint(50,750); y=random.randint(50,550)
        w=random.randint(50,150); h=random.randint(10,50)
        paredes.append(pygame.Rect(x,y,w,h))
    for _ in range(3+nivel*2):
        inimigos.append(pygame.Rect(random.randint(100,700),random.randint(100,500),25,25))
    blocos=[]
    for _ in range(4+nivel):
        blocos.append(pygame.Rect(random.randint(100,700),random.randint(100,500),30,30))
    contas_restantes=len(blocos)

def nova_pergunta():
    global pergunta,resposta
    op=random.choice(["+","-","*","/","√"])
    if op=="√":
        n=random.randint(1,20*nivel)
        pergunta=f"√{n}"; resposta=round(math.sqrt(n),2)
    else:
        a=random.randint(1,10*nivel); b=random.randint(1,10*nivel)
        if op=="/": b=random.randint(1,10*nivel)
        pergunta=f"{a}{op}{b}"; resposta=round(eval(pergunta),2)

gerar_labirinto()

while True:
    clock.tick(60); t_cor+=1
    for e in pygame.event.get():
        if e.type==pygame.QUIT: pygame.quit(); exit()
        if e.type==pygame.KEYDOWN:
            if fim and e.key==pygame.K_SPACE:
                nivel,pontos,fim,vida=1,0,False,3; gerar_labirinto()
            elif pergunta:
                if e.key==pygame.K_RETURN:
                    try:
                        if round(float(entrada),2)==resposta:
                            pontos+=1; contas_restantes-=1
                            particulas.append([player.centerx,player.centery,random.randint(3,6)])
                        else:
                            vida-=1
                            if vida<=0: fim=True
                    except:
                        vida-=1
                        if vida<=0: fim=True
                    pergunta=None; entrada=""
                elif e.key==pygame.K_BACKSPACE: entrada=entrada[:-1]
                elif e.unicode.isdigit() or e.unicode=='.': entrada+=e.unicode

    if fim:
        tela.fill(preto)
        fonte=pygame.font.SysFont(None,70)
        msg="🎉 VOCÊ VENCEU!" if nivel>5 else "💀 GAME OVER!"
        tela.blit(fonte.render(msg,True,vermelho),(W/2-200,H/2-80))
        tela.blit(fonte.render("Pressione ESPAÇO",True,branco),(W/2-250,H/2+20))
        pygame.display.flip(); continue

    keys=pygame.key.get_pressed()
    if not pergunta:
        if keys[pygame.K_LEFT]: player.x-=vel
        if keys[pygame.K_RIGHT]: player.x+=vel
        if keys[pygame.K_UP]: player.y-=vel
        if keys[pygame.K_DOWN]: player.y+=vel
    player.x=max(0,min(W-25,player.x)); player.y=max(0,min(H-25,player.y))

    for p in paredes:
        if player.colliderect(p):
            if keys[pygame.K_LEFT]: player.x+=vel
            if keys[pygame.K_RIGHT]: player.x-=vel
            if keys[pygame.K_UP]: player.y+=vel
            if keys[pygame.K_DOWN]: player.y-=vel

    for i in inimigos:
        i.x+=random.choice([-3,3]); i.y+=random.choice([-3,3])
        i.x=max(0,min(W-25,i.x)); i.y=max(0,min(H-25,i.y))
        if i.colliderect(player):
            vida-=1; player.topleft=(50,50)
            if vida<=0: fim=True

    for b in blocos[:]:
        if player.colliderect(b): nova_pergunta(); blocos.remove(b)

    if player.colliderect(porta) and not pergunta:
        if contas_restantes==0:
            nivel+=1
            if nivel>5: fim=True
            else: gerar_labirinto()
        else:
            # bloqueia passagem se não acertou todas contas
            pass

    # fundo pulsante
    r=(math.sin(t_cor*0.05)*127+128)%255
    g=(math.sin(t_cor*0.07)*127+128)%255
    b=(math.sin(t_cor*0.09)*127+128)%255
    tela.fill((r,g,b))

    pygame.draw.rect(tela,verde,porta)
    pygame.draw.rect(tela,azul,player)
    [pygame.draw.rect(tela,amarelo,b) for b in blocos]
    [pygame.draw.rect(tela,(255,random.randint(0,255),random.randint(0,255)),i) for i in inimigos]
    [pygame.draw.rect(tela,(0,0,0),p) for p in paredes]

    for part in particulas[:]:
        pygame.draw.circle(tela,(255,255,255),(part[0],part[1]),part[2])
        part[2]-=0.2
        if part[2]<=0: particulas.remove(part)

    fonte=pygame.font.SysFont(None,40)
    tela.blit(fonte.render(f"Fase:{nivel} Pontos:{pontos} Vidas:{vida}",True,branco),(10,10))

    if pergunta:
        pygame.draw.rect(tela,branco,(200,200,400,120))
        tela.blit(fonte.render(pergunta,True,preto),(330,230))
        tela.blit(fonte.render("Resposta:"+entrada,True,azul),(300,270))

    pygame.display.flip()
