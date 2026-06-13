Runner self-hosted (Windows) — passo a passo

Resumo rápido
- Objetivo: executar a job `e2e-tests` localmente usando um runner self-hosted com tag `pgats-runner`.
- Pré-requisitos: conta CircleCI, acesso ao repositório `pgats-ci`, máquina Windows com PowerShell como Administrador.

1) Gerar token no CircleCI
- Acesse: CircleCI → Project Settings (do projeto `PGATS-CI`) → Runners → Add Runner → escolha **Windows**.
- Copie o token mostrado e copie também o comando de download/registro sugerido (se fornecido).

2) Preparar a máquina Windows
- Abra PowerShell como Administrador.
- Crie diretório do runner e vá para ele:

```powershell
mkdir C:\circleci-runner
cd C:\circleci-runner
```

3) Baixar o binário do runner
- Use a URL que o CircleCI fornecer na UI (recomendado). Se não houver, use este template (substitua se a UI fornecer outra URL):

```powershell
Invoke-WebRequest -Uri "https://circleci-binaries.s3.amazonaws.com/circleci-runner/releases/latest/circleci-runner-windows-amd64.exe" -OutFile "circleci-runner.exe"
```

4) Registrar o runner no CircleCI (substitua os placeholders)
- Comando de exemplo — substitua `<TOKEN>` e `<URL>` conforme a UI:

```powershell
# Registro (exemplo)
.\circleci-runner.exe register --token "<TOKEN>" --name "pgats-runner-win-01" --tags "pgats-runner" --url "https://circleci.com"
```

- Saída esperada: confirmação e instruções; o runner aparecerá na UI do CircleCI em Project Settings → Runners.

5) Executar o runner manualmente (teste rápido)

```powershell
# Para testar sem instalar como serviço
cd C:\circleci-runner
.\circleci-runner.exe run
```

- O runner ficará conectado ao CircleCI e executará jobs atribuídos à tag `pgats-runner`.

6) Instalar o runner como serviço (recomendado) — usando `nssm`
- Baixe e extraia `nssm` e instale o serviço:

```powershell
Invoke-WebRequest -Uri "https://nssm.cc/release/nssm-2.24.zip" -OutFile "nssm.zip"
Expand-Archive nssm.zip -DestinationPath .\nssm
# Instalar serviço (ajuste paths se necessário)
.\nssm\win64\nssm.exe install CircleCI-Runner "C:\circleci-runner\circleci-runner.exe" "run"
.\nssm\win64\nssm.exe start CircleCI-Runner
```

7) Verificar o runner no CircleCI
- Em CircleCI → Project Settings → Runners, confirme que o runner aparece **online** e que tem a tag `pgats-runner`.

8) `config.yml` — já ajustado
- Confirmar que o arquivo ` .circleci/config.yml` contém `runner.tags: [pgats-runner]` (já aplicado neste repositório).

9) Testar pipeline
- Commit e push das mudanças:

```bash
git add .circleci/config.yml jest.config.cjs
git commit -m "ci: direciona job ao runner pgats-runner"
git push
```

- No CircleCI, a job deve ser atribuída ao runner local; verifique na página da pipeline e nos logs do runner (se instalado como serviço).

10) Logs, troubleshooting rápido
- Se o runner não aparece online:
  - Verifique se o processo está rodando (`.\circleci-runner.exe run`) ou se o serviço NSSM está ativo.
  - Verifique firewall/antivírus — permita conexões de saída para `circleci.com`.
  - Verifique se o token usado no registro é o mesmo mostrado na UI do CircleCI.
- Para ver logs locais, rode o runner manualmente em uma janela PowerShell e observe a saída.

11) Remover/unregister runner
- Caso precise deregistrar, na UI do CircleCI vá em Project Settings → Runners e delete o runner.
- Se instalado como serviço via `nssm`:

```powershell
.\nssm\win64\nssm.exe stop CircleCI-Runner
.\nssm\win64\nssm.exe remove CircleCI-Runner confirm
```

---

Se quiser, eu já insiro comandos exatos com o token/URL que a UI mostrar — copie aqui o token (ou o comando de registro que a UI fornecer) e eu adapto os scripts automaticamente. Se preferir, eu também posso criar um script PowerShell pronto (registrar + instalar serviço) que você executa com Admin.
