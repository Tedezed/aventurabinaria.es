---
layout: post
title: "OpenStack TripleO [Instalación]"
date: 2016-08-1 16:25:06 -0700
comments: true
---

**Laptop Overcloud**

Creamos el siguiente usuario:

    useradd -d /home/stack -m -s /bin/bash stack
    passwd stack

Configuración de sudo:
`nano /etc/sudoers`

    stack ALL=(root) NOPASSWD:ALL
    centos ALL=(root) NOPASSWD:ALL

Entramos como stack:

    su - stack

Actualizamos:

    sudo yum update

Cambiamos el nombre de la maquina:

    sudo hostnamectl set-hostname undercloud.example.com
    sudo hostnamectl set-hostname --transient undercloud.example.com

`sudo nano /etc/hosts`

    127.0.0.1         undercloud.example.com

`sudo systemctl restart network`

Comprobamos:

`[stack@localhost ~]$ hostname`

    undercloud.example.com

Repositorios necesarios:

    sudo yum -y install epel-release

Instalamos yum-plugin-priorities:

    sudo yum -y install yum-plugin-priorities

Repositorios de python-tripleoclient:

    sudo curl -o /etc/yum.repos.d/delorean-liberty.repo https://trunk.rdoproject.org/centos7-liberty/current/delorean.repo
    sudo curl -o /etc/yum.repos.d/delorean-deps-liberty.repo http://trunk.rdoproject.org/centos7-liberty/delorean-deps.repo

Instalamos Python TripleO:

    sudo yum install -y python-tripleoclient

Copiamos la configuración de ejemplo de undercloud:

    cp /usr/share/instack-undercloud/undercloud.conf.sample ~/undercloud.conf

Editamos la configuración:

`nano undercloud.conf`

    [DEFAULT]
    local_ip = 192.168.0.10/24
    undercloud_public_vip = 192.168.0.11
    undercloud_admin_vip = 192.168.0.12
    local_interface = ens8
    masquerade_network = 192.168.0.0/24
    dhcp_start = 192.168.0.60
    dhcp_end = 192.168.0.80
    network_cidr = 192.168.0.0/24
    network_gateway = 192.168.0.1
    discovery_iprange = 192.168.0.60,192.168.0.80
    [auth]

Nos aseguramos de añadir un buen DNS:

`echo nameserver 8.8.8.8 >> /etc/resolv.conf`

Instalamos undercloud:

`openstack undercloud install`

Este bug, que reporte al launchpad esta actualmente solucionado.

**ERROR:** https://bugs.launchpad.net/tripleo/+bug/1544150

    1710 packages excluded due to repository priority protections
    Traceback (most recent call last):
        File "<string>", line 1, in <module>
        File "/usr/lib/python2.7/site-packages/instack_undercloud/undercloud.py", line 750, in install
            _run_instack(instack_env)
        File "/usr/lib/python2.7/site-packages/instack_undercloud/undercloud.py", line 636, in _run_instack
            _run_live_command(args, instack_env, 'instack')
        File "/usr/lib/python2.7/site-packages/instack_undercloud/undercloud.py", line 341, in _run_live_command
            line = process.stdout.readline().decode()
    UnicodeDecodeError: 'ascii' codec can't decode byte 0xc3 in position 74: ordinal not in range(128)

**SOLUCION:**

`nano /usr/lib/python2.7/site-packages/instack_undercloud/undercloud.py`

    line = process.stdout.readline().decode('utf-8')

Finalizado del comando

    --------------------- END PROFILING ---------------------
    [2016-03-14 14:15:35,119] (os-refresh-config) [INFO] Completed phase post-configure
    os-refresh-config completed successfully
    Generated new ssh key in ~/.ssh/id_rsa
    Created flavor "baremetal" with profile "None"
    Created flavor "control" with profile "control"
    Created flavor "compute" with profile "compute"
    Created flavor "ceph-storage" with profile "ceph-storage"
    Created flavor "block-storage" with profile "block-storage"
    Created flavor "swift-storage" with profile "swift-storage"

    #############################################################################
    Undercloud install complete.

    The file containing this installation's passwords is at
    /home/stack/undercloud-passwords.conf.

    There is also a stackrc file at /home/stack/stackrc.

    These files are needed to interact with the OpenStack services, and should be
    secured.

    #############################################################################



**ERROR LOOP al apagar:** https://bugzilla.redhat.com/show_bug.cgi?id=1178497
`rm: cannot remove /lib/drauct/hooks/shutdown/30-dm-shutdown.sh: Read-only filesystem`

**SOLUCIÓN:**

Reinstalamos Dracut:

    sudo yum reinstall dracut

Editamos:

    sudo nano /usr/lib/dracut/modules.d/99shutdown/shutdown.sh

Despues de:

    . /lib/dracut-lib.sh

Añadir:

    if [ "$(stat -c '%T' -f /)" = "tmpfs" ]; then
            mount -o remount,rw /
    fi

Editamos:

`sudo nano /usr/lib/dracut/modules.d/99shutdown/module-setup.sh`

Buscar:

    inst_multiple umount poweroff reboot halt losetup

Cambiar por:

    inst_multiple umount poweroff reboot halt losetup stat

Recrear initramfs:

    sudo dracut --force

unmask shutdown:

    sudo systemctl unmask dracut-shutdown.service

Reinciamos:

    sudo reboot

Activamos stackrc con:

    source /home/stack/stackrc

Creación de imagenes para el opvercloud:

* **Opcion 1:**

    Creamos las imagenes, puede tardar mucho tiempo:
        
        openstack overcloud image build --all

    **ERROR** `Required file "ironic-python-agent.initramfs" does not exist.`

    **SOLUCIÓN**

    La intuición me dice que los dos ficheros son el mismo (Comparando .kernel de fedora)
        ```
        [stack@undercloud images2]$ ls -hl *kernel
        -rw-r--r--. 1 stack stack 5,0M mar 16 08:24 deploy-ramdisk-ironic.kernel
        -rw-r--r--. 1 stack stack 5,0M mar 16 08:25 ironic-python-agent.kernel
        [stack@undercloud images2]$ diff *kernel
        cp deploy-ramdisk-ironic.initramfs ironic-python-agent.initramfs
        cp deploy-ramdisk-ironic.kernel ironic-python-agent.kernel
        ```

* **Opcion 2:**

    Bajar las imagenes de fedora:

    `wget -r -nd -np --reject "index.html*" https://repos.fedorapeople.org/repos/openstack-m/rdo-images-centos-liberty-opnfv/`

Subimos las imagenes al undercloud con:

    openstack overcloud image upload --image-path /home/stack/images

**BONUS** - borrar imagenes de glance:

    for i in $(glance image-list | grep -v ID | awk ' { print $2 } '); do glance image-delete $i; done

Podremos ver las imagenes cargadas con:

    [stack@undercloud images]$ openstack image list
    +--------------------------------------+------------------------+
    | ID                                                                     | Name                                     |
    +--------------------------------------+------------------------+
    | f872cf08-afd7-4d86-a008-465730c3ecb2 | bm-deploy-kernel             |
    | b3215dfe-8548-42b0-80aa-2ccae02574ac | bm-deploy-ramdisk            |
    | 583c3a7f-bce3-47db-ba0d-0750e601684f | overcloud-full                 |
    | 1f2166b1-e8bc-4351-b5db-8ebcad37b9f2 | overcloud-full-initrd    |
    | f327bae7-3d53-4a78-be14-2139457d04ad | overcloud-full-vmlinuz |
    +--------------------------------------+------------------------+

Podremos ver las subredes con:

    [stack@undercloud images]$ neutron subnet-list
    +--------------------------------------+------+----------------+--------------------------------------------------+
    | id                                                                     | name | cidr                     | allocation_pools                                                                 |
    +--------------------------------------+------+----------------+--------------------------------------------------+
    | 791d86a8-e607-4f67-9c85-b5228f599910 |            | 192.168.0.0/24 | {"start": "192.168.0.60", "end": "192.168.0.80"} |
    +--------------------------------------+------+----------------+--------------------------------------------------+

Actualizamos la red con el dns:

`neutron subnet-update 791d86a8-e607-4f67-9c85-b5228f599910 --dns-nameserver 192.168.0.81`

    [stack@undercloud ~]$ neutron subnet-show 791d86a8-e607-4f67-9c85-b5228f599910
    +-------------------+------------------------------------------------------------------+
    | Field                         | Value                                                                                                                        |
    +-------------------+------------------------------------------------------------------+
    | allocation_pools    | {"start": "192.168.0.60", "end": "192.168.0.80"}                                 |
    | cidr                            | 192.168.0.0/24                                                                                                     |
    | dns_nameservers     | 8.8.4.4                                                                                                                    |
    | enable_dhcp             | True                                                                                                                         |
    | gateway_ip                | 192.168.0.1                                                                                                            |
    | host_routes             | {"destination": "169.254.169.254/32", "nexthop": "192.168.0.81"} |
    | id                                | 791d86a8-e607-4f67-9c85-b5228f599910                                                         |
    | ip_version                | 4                                                                                                                                |
    | ipv6_address_mode |                                                                                                                                    |
    | ipv6_ra_mode            |                                                                                                                                    |
    | name                            |                                                                                                                                    |
    | network_id                | 67917664-1468-4658-bba4-90574e1bd997                                                         |
    | subnetpool_id         |                                                                                                                                    |
    | tenant_id                 | 7b4c8e02a48849aaa8a0d06bb44a4f8c                                                                 |
    +-------------------+------------------------------------------------------------------+

En mi caso a los clientes no les asigna el DNS, podemos introducirlo manualmente:
`echo server=8.8.8.8 >> /etc/ironic-inspector/dnsmasq.conf`

**BONUS** - Eliminar las maquinas
    
    for i in {1..2}; do virsh destroy overcloud-node$i; virsh undefine overcloud-node$i; done

Configuramos el acceso:

    sudo cat << EOF > /etc/polkit-1/localauthority/50-local.d/50-libvirt-user-stack.pkla
    [libvirt Management Access]
    Identity=unix-user:centos
    Action=org.libvirt.unix.manage
    ResultAny=yes
    ResultInactive=yes
    ResultActive=yes

Añadimso la clave publica para la conexión ssh:
`nano .ssh/authorized_keys`



Anfitrion Undercloud
--------------------

    cat << EOF > /usr/bin/bootif-fix
    #!/usr/bin/env bash

    while true;
                    do find /httpboot/ -type f ! -iname "kernel" ! -iname "ramdisk" ! -iname "*.kernel" ! -iname "*.ramdisk" -exec sed -i 's|{mac|{net0/mac|g' {} +;
    done
    EOF

`chmod a+x /usr/bin/bootif-fix`

    cat << EOF > /usr/lib/systemd/system/bootif-fix.service
    [Unit]
    Description=Automated fix for incorrect iPXE BOOFIF

    [Service]
    Type=simple
    ExecStart=/usr/bin/bootif-fix

    [Install]
    WantedBy=multi-user.target
    EOF 

`chmod a+x /usr/lib/systemd/system/bootif-fix.service`

    systemctl daemon-reload
    systemctl enable bootif-fix
    systemctl start bootif-fix




Overcloud
---------

Mapeamos las maquinas con el nombre overcloud-node* y optenemos la MAC que tienen en el puente aprovisionamient:

    for i in {1..2}; do virsh -c qemu+ssh://xerrot@192.168.0.82/system domiflist overcloud-node$i | awk '$3 == "interno" {print $5};'; done > /home/stack/nodes.txt

BONUS - Borrar todos los nodos de ironic:

    for i in $(ironic node-list | grep -v UUID | awk ' { print $2 } '); do ironic node-delete $i; done

BONUS - Apagar los nodos:

    for i in $(ironic node-list | grep -v UUID | awk ' { print $2 } '); do ironic node-set-power-state $i off; done

Introducir nodos:

    jq . << EOF > /home/stack/instackenv.json
    {
        "ssh-user": "xerrot",
        "ssh-key": "$(cat cat /home/stack/.claves/id_rsa)",
        "power_manager": "nova.virt.baremetal.virtual_power_driver.VirtualPowerManager",
        "host-ip": "192.168.0.81",
        "arch": "x86_64",
        "nodes": [
            {
                "pm_addr": "192.168.0.82",
                "pm_password": "$(cat /home/stack/.claves/id_rsa)",
                "pm_type": "pxe_ssh",
                "mac": [
                    "$(sed -n 1p /home/stack/nodes.txt)"
                ],
                "cpu": "4",
                "memory": "4096",
                "disk": "45",
                "arch": "x86_64",
                "pm_user": "xerrot"
            },
            {
                "pm_addr": "192.168.0.82",
                "pm_password": "$(cat /home/stack/.claves/id_rsa)",
                "pm_type": "pxe_ssh",
                "mac": [
                    "$(sed -n 2p /home/stack/nodes.txt)"
                ],
                "cpu": "4",
                "memory": "4096",
                "disk": "45",
                "arch": "x86_64",
                "pm_user": "xerrot"
            }
        ]
    }
    EOF

Importamos la configuración anterior:

    openstack baremetal import --json instackenv.json

Podremos listar los nodos con:

    ironic node-list

Configuramos el boot:

    openstack baremetal configure boot

Realizamos una introspection:

    openstack baremetal introspection bulk start

**ERROR** en maquina virtual http://ipxe.org/040ee119

**SOLUCION:**

Reducimos el tiempo de espera:

    sudo brctl setfd br1 2

Consultar log con:

    sudo journalctl -fu openstack-ironic-inspector-dnsmasq -fu openstack-ironic-inspector 

Consultar puertos:

    sudo tcpdump -i any port 67 or port 68 or port 69 or port 80 or port 8088


**ERROR** http://ipxe.org/2e008001 o errores en el arranque de las maquinas:

    agent.kernel y agent.ramdisk en blanco o dañados.

**SOLUCION:**

    cd /httpboot
    sudo cp -f ironic-python-agent.kernel /httpboot/agent.kernel
    sudo cp -f ironic-python-agent.initramfs /httpboot/agent.ramdisk

Asignamos rol:

    ironic node-list

    ironic node-update a04bddb8-2dc3-4627-8c0b-82a02a574a86 replace properties/capabilities=profile:control,boot_option:local
    ironic node-update 26fc0ce0-d6dd-42ae-81c7-e9e0e3d5dcb5 replace properties/capabilities=profile:compute,boot_option:local

Validamos con:

    openstack overcloud deploy --templates --control-scale 1 --compute-scale 1 --neutron-tunnel-types vxlan --neutron-network-type vxlan \
        --validation-errors-fatal --validation-warnings-fatal --dry-run 

    openstack baremetal configure ready state
    openstack baremetal instackenv validate

Desplegamos overcloud ejecutando:

    openstack overcloud deploy --templates --control-scale 1 --compute-scale 1