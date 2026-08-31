# lotus
Shared assets among TUIs

Install themes into `~/.config/lotus/`:
```
mkdir -p ~/.config/lotus
cd ~/.config/lotus
git clone --filter=blob:none --no-checkout https://github.com/lotus-io/lotus.git .
git sparse-checkout init --no-cone
git sparse-checkout set 'themes/**'
git checkout
rm -rf .git
```