# Ansible Role: Dotfiles

Installs configuration files (dotfiles) by creating symlink files in user
specified locations that reference the files in a user specified directory
(typically a cloned repository).  By default, it installs [my
dotfiles](https://github.com/JerryShockley/dotfiles), but it can be used with
any dotfiles if configured properly.

###### XDG Base Directory Standard

This script supports the [XDG Base Directory
Standard](https://standards.freedesktop.org/basedir-spec/basedir-spec-latest.html).
It recognizes the XDG environment variables _XDG_CONFIG_HOME_ and
_XDG_DATA_HOME_ along with their default values if the environment variables are
undefined.

###### Filename Collisions

Any prexisting non-symlink files that result in a name collision are preserved.
They are backed up to the specified directory in the options shown below. This
directory should be empty if it exists and will be created if it does not exist.
Note we do not back up symlink files as they refer to a file that exists in
another location and that file is unchanged after we remove the symlink file. 

## Requirements

Requires a directory of dotfiles(typically a cloned dotfiles directory). 

## Role Variables

Shown below are the varibles used by this role along with default values. To
override these default values simply assign one or more of the variables below
to a different value in the playbook you use this role in.


Home directory of current user. Typically, the location of configuration files
for applications that do not follow the XDG Base Directory Standard.

    home: "{{ ansible_env.HOME | default('~') }}"

The directory the dotfiles repo is cloned into. This directory will be created
if it doesn't already exist. If the specified directory does exist, it should be
  empty.

    dotfiles_local_repo_dir: "{{ ansible_env.home }}/.dotfiles"

The directory used to backup any preexisting non-symlink configuration files
found in the target path for a symlink file (filename collision). This directory
will be created if it does not already exist. We strongly recommend that the
directory is empty if it does already exist to avoid secondary collisions.

    dotfiles_backup: "{{ home }}/.dotfiles_backup"

The location of xdg_config_home for application config files that follow the xdg
base directory standard. Note the application name needs to be appended to this
home directory path. This directory will be created if it doesn't already exist.


    xdg_config: "{{ ansible_env.xdg_config_home | default( home + '/.config' )
    }}"

The location of XDG_DATA for application data files that follow the XDG Base
Directory Standard. Note the application name needs to be appended to this home
directory path.  

    xdg_data: "{{ ansible_env.XDG_DATA_HOME |\ default( home + '/.local/share' )
    }}"

Configuration directory for Neovim

    nvim_config: "{{ xdg_config }}/nvim"

Data directory for Neovim

    nvim_data: "{{ xdg_data }}/nvim"

List of directories to be created if they do not already exist.  All
intermediate directories will be created if they do not already exist as well.
For example, if you are targeting symlinks such as _dir1/dir2/dir3/myfile.vim_
and _dir1/otherfile.vim_j you need only one directory entry: _"{{ dir1/dir2/dir3
}}"_. This entry creates 3 nested directories such that _"{{ dir1 }}"_ would be
redundant: 

    - dir1 
      - dir2 
        -  dir3

All directories specified by _dotfiles_ below, must be listed here.

    directories:
      - "{{ dotfiles_backup }}"
      - "{{ nvim_data }}/site/plugged/snippets"
      - "{{ nvim_config }}/after/plugin"
      - "{{ nvim_config }}/ftplugin"

List of configuration files to be installed and the target directories where the
symlink files are created. These symlink files point back to the corresponding
source files specified by _dotfiles_local_repo_. 

  - _src_file_: Relative file path from both _dotfiles_local_repo_dir_ and
    _target_dir_. Such that the link source file (repository) is:
    _dotfiles_local_repo_dir_/_src_file_ and the symlink file path is:
    _target_dir_/_src_file_.

  - _target_dir_: Root directory for remote symlink file creation. This
    directory should mirror _dotfiles_local_repo_dir_ relative to _src_file_
    subdirectory location.

Note when using XDG directory paths in _target_dir_ that the application name
should be appended to the XDG directory. Such as the value of _nvim_config_ has
_nvim_ appended.

    dotfiles:
      - { src_file: '.zshrc', target_dir: "{{ home }}" }
      - { src_file: '.zshenv', target_dir: "{{ home }}" }
      - { src_file: '.zprofile', target_dir: "{{ home }}" }
      - { src_file: '.tmux.conf', target_dir: "{{ home }}" }
      - { src_file: '.gitignore', target_dir: "{{ home }}" }
      - { src_file: '.gemrc', target_dir: "{{ home }}" }
      - { src_file: '.zshrc', target_dir: "{{ home }}" }
      - { src_file: 'init.vim', target_dir: "{{ nvim_config }}" }
      - { src_file: 'commands.vim', target_dir: "{{ nvim_config }}" }
      - { src_file: 'plugins.vim', target_dir: "{{ nvim_config }}" }
      - { src_file: 'key_mappings.vim', target_dir: "{{ nvim_config }}" }
      - { src_file: 'coc-settings.json', target_dir: "{{ nvim_config }}" }
      - { src_file: 'after/plugin/emmet.vim', target_dir: "{{ nvim_config }}" }
      - { src_file: 'ftplugin/denite.vim', target_dir: "{{ nvim_config }}" }
      - { src_file: 'ftplugin/denite-filter.vim', target_dir: "{{ nvim_config
        }}" }

## Dependencies

None

## Example Playbook

    - hosts: localhost roles:
        - role: jerryshockley.dotfiles vars: dotfiles_local_repo_dir:
          ~/.dotfiles

## License

mit

## Author Information

Created in 2018 by Jerry Shockley.
