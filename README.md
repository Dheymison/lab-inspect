# LAB 2 — Inspeção e Troubleshooting (Docker do dia a dia)

## Objetivo
Treinar o “modo operador” no Docker: subir um container, **inspecionar**, **diagnosticar** e **corrigir** problemas comuns usando comandos do dia a dia.

Você vai praticar:
- `docker ps`, `docker logs`, `docker exec`, `docker top`, `docker inspect`, `docker stats`
- Identificar “subiu mas não responde”
- Identificar “subiu e caiu”
- Corrigir sem precisar rebuildar imagem

**Tempo sugerido:** 30–45 min.

---

## Pré-requisitos
Docker funcionando na VM Debian:
```bash
docker version
```

---

## Checkpoint A — Subir um Nginx oficial
1) Suba o container:
```bash
docker rm -f lab2-web 2>/dev/null || true
docker run -d --name lab2-web -p 8080:80 nginx:alpine
```

2) Valide no host:
```bash
curl http://localhost:8080
```

**Critério de sucesso:** resposta `HTTP/1.1 200 OK` (ou similar).

---

## Checkpoint B — Inspeção básica
1) Ver status:
```bash
docker ps
```

2) Ver logs:
```bash
docker logs --tail 50 lab2-web
```

3) Ver processos do container:
```bash
docker top lab2-web
```

4) Ver detalhes (portas, IP, mounts):
```bash
docker inspect lab2-web | less
```

5) Ver consumo de recursos:
```bash
docker stats --no-stream
```

---

## Cenário 1 — “Subiu, mas a página sumiu / retornou erro”
### Ação (simular problema)
Apague o conteúdo do document root:

```bash
docker exec -it lab2-web sh -c "rm -rf /usr/share/nginx/html/*"
```

### Sintoma
```bash
curl http://localhost:8080 | head -n 20
```
Você deve ver erro (404/403) ou página vazia.

### Diagnóstico (o que olhar)
Entre no container e inspecione:
```bash
docker exec -it lab2-web sh
ls -lah /usr/share/nginx/html
exit
```

### Correção (sem rebuild)
Crie um HTML simples de volta:
```bash
docker exec -it lab2-web sh -c 'cat > /usr/share/nginx/html/index.html <<"EOF"
<h1>Recuperado no LAB 2</h1>
<p>Corrigi sem rebuild: usei docker exec.</p>
EOF'
```

Valide novamente:
```bash
curl -sS http://localhost:8080
```

---

## Cenário 2 — “Subiu e caiu (processo principal morreu)”
### Ação (simular problema)
Mate o processo do Nginx:
```bash
docker exec -it lab2-web sh -c "nginx -s stop || true"
```

### Sintoma
O container pode parar:
```bash
docker ps -a | grep lab2-web
```

### Diagnóstico
Veja status + logs:
```bash
docker ps -a
docker logs lab2-web
```

### Correção
Reinicie o container:
```bash
docker start lab2-web
```

Valide:
```bash
curl http://localhost:8080
```

---

## Evidências (para entrega)
Tire prints (ou cole o output) de:
1) `docker ps` com `lab2-web`
2) `docker logs --tail 20 lab2-web`
3) `curl http://localhost:8080` (antes e depois de corrigir um cenário)

---

## Troubleshooting rápido
### “bind: address already in use” na porta 8080
Descubra quem está usando:
```bash
sudo ss -lntp | grep :8080
```
Escolha outra porta:
```bash
docker rm -f lab2-web
docker run -d --name lab2-web -p 8081:80 nginx:alpine
curl http://localhost:8081
```

---

## Bônus (para quem terminar antes)
1) Mostre o IP interno do container:
```bash
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' lab2-web
```

2) Compare `docker logs` vs `docker exec` (o que vai para logs e o que não vai).
