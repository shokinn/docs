# MkDocs

## Setup local dev environment

```bash
MKDOCS_VERSION="1.0.4" && \
MKINCLUDE_VERSION="0.5.1" && \
PYMD_EXT_VERSION="5.0" && \
PYGMENTS_VERSION="2.2.0" && \
MATERIAL_VERSION="3.0.4" && \
sudo apt update && \
sudo apt install -y python3 python3-pip && \
sudo -H pip3 install --upgrade pip && \
sudo -H pip3 install \
  mkdocs==${MKDOCS_VERSION} \
  markdown-include==${MKINCLUDE_VERSION} \
  pymdown-extensions==${PYMD_EXT_VERSION} \
  pygments==${PYGMENTS_VERSION} \
  mkdocs-material==${MATERIAL_VERSION}
```