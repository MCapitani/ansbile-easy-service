# Easy-service

Ansible role to create a new systemd service running a given executable.
Takes care of deploying the executable and configuring the service providing sensible defaults, allowing minimal configuration.

## Requirements

Only usable on systemd-using targets.

## Role Variables

- `unit_file_folder`:
  Remote folder which should contain the unit file.
  Default: `/lib/systemd/system/`.
- `bin_dest`:
  Remote path for the service executable file(s).
  Uses `copy` module syntax together with `bin_src`.
  Default: `/usr/share`.
- `config_dest`:
  Remote path for the service configuration file(s).
  Uses `copy` module syntax together with `config_src`.
  Default: `/etc/<service-name>`.
- `unit_file_name`:
  Name to give to the unit file.
  Default: `<service-name>.service`.
- `bin_src`:
  File or folder containing the executable file(s) on the local host.
  Uses `copy` module syntax together with `bin_dest`.
  Mandatory.
- `config_src`:
  Configuration file or folder on the local host.
  Uses `copy` module syntax together with `config_dest`.
  Default: no configuration (nothing will be copied on the remote).
- `config_templates`:
  List of templates to be used to generate configuration files.
  Will behave exatly like `config_src` except that the files
  will be parsed as jinja templates. The mentioned variables should
  be present in the scope (e.g. declared in the playbook, passed with `-e`
  or passed as role variables through `vars:`)
  Default: no templates

- `service_name`:
  The name of the service.
  Mandatory.
- `service_command`:
  The command which should be run to start the service.
  Default: Run `bin_dest` if it is a file, otherwise run `bin_dest + basename(bin_src)`. This default will not work if `bin_src` is a directory.
- `service_user`:
  The (linux) user to be used to run the command.
  If not present on the target system, it will be created.
  Default: `root`.
- `service_working_dir`:
  The working directory to be used to run the command.
  Default: `bin_dest` if it is a directory, otherwise its containing folder.
- `service_state`:
  The required state the service. Either `started`, `restarted`, `reloaded` or `stopped`.
  Default: `started`.
- `service_enabled`:
  If the service should be set to be run at system boot.
  Default: `yes`.
- `service_restart`:
  In what cases should the executable be restarted (as the systemd unit-file parameter with the same name).
  Default: `on-failure`.

Please note that the remote folders and files `*_dest` will be set as owned by `service_user`. Take care not to use the default ones with a non-`root` user, as that will lead to errors or problematic behaviour.

## Dependencies

None.

## Example Playbook

Creating a service which simply runs a binary is extremely simple:

    - name: Install simple service
      become: yes
      become_user: root
      import_role:
        name: nextworks.easy-service
      vars:
        service_name: my-service
        bin_src: "{{ bin_local_path }}"

nonetheless, there are several configuration options available:

    - name: Install service with configuration
      become: yes
      become_user: root
      import_role:
        name: nextworks.easy-service
      vars:
        service_name: my-complex-service
        service_user: mcs-user
        bin_src: "{{ bin_local_path }}"
        bin_dest: /usr/share/mcs/mcs-bin
        config_src: files/aux.conf
        config_dest: /etc/mcs/
        config_templates:
          - src: templates/my.conf.j2
            dest: /etc/mcs/mcs.conf
        service_command: "/usr/share/mcs/mcs-bin -c /etc/mcs/mcs.conf --aux-conf=/etc/mcs/aux.conf"

## License

Apache-v2.0 (https://www.apache.org/licenses/LICENSE-2.0)

## Author Information

Marco Capitani (m.capitani AT nextworks DOT it).
Copyright 2019, Nextworks s.r.l (www.nextworks.it)
