# FEE-Control-Panel for developers

The front-end is currently a web-based UI made using vue.js. For developers, please follow the below instructions to get the UI running locally.

### Step 1. First you will need node.js installed in your machine. 
If you're using homebrew with macOS for example, you can do
```
brew install nvm
mkdir ~/.nvm
# Add these to your shell rc (zsh):
echo 'export NVM_DIR="$HOME/.nvm"' >> ~/.zshrc
echo '[ -s "/usr/local/opt/nvm/nvm.sh" ] && \. "/usr/local/opt/nvm/nvm.sh"' >> ~/.zshrc
source ~/.zshrc

nvm install --lts
nvm use --lts
```
Should be similar for other operating systems as well.

### Step 2. Clone and run the Vue.js app
```
git clone https://github.com/spadi-alliance/FEE-Control-Software.git
cd devel/front-end-ui
npm install
npm run dev
```
Then simple open the app in your browser, for example using `http://localhost:5173/FEEctrlSW/`

- You can do `q+enter` quit the app in the terminal.
- Note: No need to run `git clone --recursive https://github.com/spadi-alliance/FEE-Control-Software.git` if you only need to clone the devel files.