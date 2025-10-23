# 🧩 Script de Descriptografia de Arquivos com Fernet

Este script descriptografa todos os arquivos `.enc` ou `.txt` dentro da pasta `teste_files`, utilizando a chave armazenada em `chave.key`.

---

## 📜 Código Python

```python
#!/usr/bin/env python3
from cryptography.fernet import Fernet, InvalidToken
import os
import sys

PASTA = "teste_files"
NOME_CHAVE = "chave.key"

def carregar_chave():
    # tenta carregar chave da pasta atual
    if os.path.isfile(NOME_CHAVE):
        with open(NOME_CHAVE, "rb") as f:
            return f.read()
    # tenta carregar chave dentro da pasta alvo
    caminho2 = os.path.join(PASTA, NOME_CHAVE)
    if os.path.isfile(caminho2):
        with open(caminho2, "rb") as f:
            return f.read()
    raise FileNotFoundError(f"Arquivo de chave '{NOME_CHAVE}' não encontrado na pasta atual nem em '{PASTA}'.")

def descriptografar_arquivo(caminho_arquivo, fernet):
    with open(caminho_arquivo, "rb") as f:
        dados_criptografados = f.read()
    # tenta descriptografar (pode lançar InvalidToken)
    dados_descriptografados = fernet.decrypt(dados_criptografados)
    return dados_descriptografados

def nome_saida(orig_path):
    # se terminar com .enc, remove .enc
    if orig_path.endswith(".enc"):
        return orig_path[:-4]
    # caso contrário, adiciona sufixo .decrypted para não sobrescrever
    return orig_path + ".decrypted"

def main():
    if not os.path.isdir(PASTA):
        print(f"Erro: pasta '{PASTA}' não encontrada. Coloque a pasta no mesmo diretório do script ou ajuste a variável PASTA.")
        sys.exit(1)

    try:
        chave = carregar_chave()
    except FileNotFoundError as e:
        print("Erro:", e)
        sys.exit(1)

    fernet = Fernet(chave)

    arquivos = os.listdir(PASTA)
    candidatos = [f for f in arquivos if f.lower().endswith((".enc", ".txt")) and f != NOME_CHAVE]

    if not candidatos:
        print(f"Nenhum arquivo .enc ou .txt encontrado em '{PASTA}'.")
        sys.exit(0)

    for fname in candidatos:
        caminho = os.path.join(PASTA, fname)
        try:
            dados = descriptografar_arquivo(caminho, fernet)
        except InvalidToken:
            print(f"❌ '{fname}': chave inválida ou arquivo corrompido (InvalidToken).")
            continue
        except Exception as e:
            print(f"❌ '{fname}': falha ao ler/descriptografar -> {e}")
            continue

        out_name = nome_saida(caminho)
        try:
            with open(out_name, "wb") as out_f:
                out_f.write(dados)
            print(f"✅ '{fname}' descriptografado com sucesso -> '{out_name}'")
        except Exception as e:
            print(f"❌ erro ao gravar '{out_name}': {e}")

if __name__ == "__main__":
    main()
