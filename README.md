code: 

#!/usr/bin/env bash
# Hundunix 0.1 — Stage 0: safe LFS build workspace preparation
# This script does NOT repartition disks, format filesystems, install packages,
# change the host OS, or create a fake ISO.

set -Eeuo pipefail

PROJECT="Hundunix"
VERSION="0.1"
WORK="${HUNDUNIX_WORK:-$HOME/hundunix-build}"
BOOK_URL="https://linuxfromscratch.org/lfs/downloads/stable-systemd/LFS-BOOK-13.1-NOCHUNKS.html"

say() { printf '\n[HUNDUNIX] %s\n' "$*"; }
fail() { printf '\n[ERRO] %s\n' "$*" >&2; exit 1; }

command -v uname >/dev/null 2>&1 || fail "Comando uname não encontrado."
command -v mkdir >/dev/null 2>&1 || fail "Comando mkdir não encontrado."

say "Hundunix ${VERSION} — preparação inicial"
say "Arquitetura detectada: $(uname -m)"
say "Kernel do computador anfitrião: $(uname -sr)"

case "$(uname -m)" in
  x86_64) say "Arquitetura x86_64 confirmada." ;;
  *) fail "Este primeiro projeto está definido para x86_64. Nenhuma alteração foi feita." ;;
esac

for cmd in bash gcc g++ make awk bison gawk m4 patch perl python3 tar xz gzip wget curl; do
  if command -v "$cmd" >/dev/null 2>&1; then
    printf '  [OK] %s\n' "$cmd"
  else
    printf '  [FALTA] %s\n' "$cmd"
  fi
done

mkdir -p "$WORK"/{sources,build,logs,rootfs,iso,docs}
say "Diretórios de trabalho criados em: $WORK"

if command -v curl >/dev/null 2>&1; then
  say "Baixando o livro oficial LFS 13.1 (documentação; não é o sistema compilado)..."
  if curl --fail --location --retry 2 "$BOOK_URL" -o "$WORK/docs/LFS-BOOK-13.1.html"; then
    say "Livro LFS salvo em $WORK/docs/LFS-BOOK-13.1.html"
  else
    say "Não consegui baixar o livro agora. O espaço de trabalho continua pronto."
  fi
else
  say "curl não está instalado; pulei o download da documentação."
fi

{
  echo "Project=${PROJECT}"
  echo "Version=${VERSION}"
  echo "Architecture=$(uname -m)"
  echo "HostKernel=$(uname -sr)"
  echo "Created=$(date -Is 2>/dev/null || date)"
  echo "Workspace=${WORK}"
} > "$WORK/HUNDUNIX-BUILD-INFO.txt"

cat <<EOF

============================================================
 HUNDUNIX 0.1 — ETAPA 0 CONCLUÍDA
============================================================
Pasta: $WORK

IMPORTANTE:
- Nenhuma partição foi modificada.
- Nenhum sistema de arquivos foi formatado.
- Nenhuma ISO foi criada nesta etapa.
- O próximo passo é conferir os pré-requisitos do host e
  seguir a sequência exata do livro LFS 13.1.
- Faça a compilação em uma máquina virtual/disco virtual,
  nunca na partição do seu sistema principal.

Quando terminar, envie o conteúdo de:
  $WORK/HUNDUNIX-BUILD-INFO.txt
e a lista dos itens [FALTA] que apareceram acima.
============================================================
EOF