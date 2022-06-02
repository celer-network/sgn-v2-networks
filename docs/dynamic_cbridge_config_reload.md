# Dynamic cBridge config reload

Previously, adding, deleting and updating chains are done by editing `cbridge.toml` and restarting `sgnd`.
Since v1.9.1, `sgnd` supports dynamic config reload via `SIGHUP`.

1. Find out the PID of `sgnd`. If run via systemd, use `systemctl`:

    ```sh
    systemctl status sgnd
    ```

    Example output:

    ```
    ● sgnd.service - SGN daemon
        Loaded: loaded (/etc/systemd/system/sgnd.service; enabled; vendor preset: enabled)
        Active: active (running) since Tue 2022-05-31 09:11:06 UTC; 1h 2min ago
    Main PID: 1480949 (sgnd)
        Tasks: 10 (limit: 9303)
        Memory: 3.9G
        CGroup: /system.slice/sgnd.service
                └─1480949 /home/ubuntu/go/bin/sgnd start --home /home/ubuntu/.sgnd/
    ```

2. Send `SIGHUP` to `sgnd`:

    ```sh
    kill -s HUP 1480949
    ```

3. Alternatively, if `sgnd` is run via Docker (with pid 1), send `SIGHUP` to Docker using `docker-compose`:

    ```sh
    docker-compose kill -s SIGHUP [docker-name]
    ```
