## Version management
- Seems like the most straightforward way to manage this is with `pyenv`
    - found these helpful:
        - https://realpython.com/intro-to-pyenv/
        - https://stackoverflow.com/a/68228627

- in $HOME/.bash_profile, make sure the following is at top
    ```
    if command -v pyenv 1>/dev/null 2>&1; then
        eval "$(pyenv init --path)"
    fi
    ```
    - default might be something like `eval "$(pyenv init -)"` and it doesn't work for actually changing python settings

- `pyenv install --list` shows all available packages
- install specific version with `pyenv install [version]` 
    - eg. `pyenv install 3.8.0`

- `pyenv verisons` shows available packages
    - current global one is starred
    
- `pyenv which python` shows executable for current global version
- to set global version `pyenv global [version, eg. 3.8.0]`
    - now `python --version` will show 3.8.0

- use this to create virtualenv with specific version
    - `python -m venv .venv`
    - in activated venv, version will correspond to pyenv global version

- `pyenv global system` will set to system default, 2.7.x for example

- `python3` will still use whatever version executed at `which python3`
- shouldn't affect version in existing virtualenvs
