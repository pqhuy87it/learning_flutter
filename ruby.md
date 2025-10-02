Install Homebrew
Install rbenv and ruby-build: ruby-build is a plugin for rbenv that helps compile and install Ruby versions.
brew install rbenv ruby-build
edit zshrc, add eval "$(rbenv init -)" or terminal: echo 'eval "$(rbenv init -)"' >> ~/.zshrc
rbenv install 3.4.1
rbenv global 3.4.1 
