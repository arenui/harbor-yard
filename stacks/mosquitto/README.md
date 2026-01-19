# Mosquitto MQTT Broker

Eclipse Mosquitto is an open-source MQTT broker for IoT messaging.

## Configuration Files

Configuration files are managed as SOPS-encrypted files in `private/configs/`:

- `mosquitto-mosquitto.conf` - Main broker configuration
- `mosquitto-passwd` - Password file with user credentials
- `mosquitto-acl` - Access control list (optional)

These files are automatically decrypted and deployed to `/srv/stacks/mosquitto/` by the `secrets` Ansible role during deployment.

## Password File Management

The `mosquitto-passwd` file contains hashed passwords. To update it:

### Generate Password Hashes

```bash
# Generate a new password hash
docker run --rm eclipse-mosquitto mosquitto_passwd -b -c /dev/stdout username password

# This outputs: username:$7$101$hash...
```

### Update the Encrypted Password File

```bash
# Edit the encrypted password file
SOPS_AGE_KEY_FILE=~/.config/sops/age/keys.txt sops private/configs/mosquitto-passwd

# Add/update users (one per line):
mqtt-admin:$7$101$hash1...
homeassistant:$7$101$hash2...
zigbee2mqtt:$7$101$hash3...
zwave-js-ui:$7$101$hash4...
```

### Apply Changes

After updating the password file, redeploy:

```bash
ansible-playbook -i private/inventory.yml public/site.yml -l <hostname>
docker restart mosquitto
```

## Configuration

### mosquitto.conf

Main configuration includes:

- Persistence and logging settings
- MQTT listener on port 1883
- WebSockets listener on port 9001
- Authentication enabled (no anonymous access)
- Password file location
- Optional ACL configuration

To edit:

```bash
SOPS_AGE_KEY_FILE=~/.config/sops/age/keys.txt sops private/configs/mosquitto-mosquitto.conf
```

### acl (Access Control List)

Fine-grained topic permissions. Default allows all authenticated users full access.

Example ACL rules:

```
user homeassistant
topic readwrite #

user zigbee2mqtt
topic readwrite zigbee2mqtt/#

user sensor-device
topic write sensors/#
topic read commands/#
```

To edit:

```bash
SOPS_AGE_KEY_FILE=~/.config/sops/age/keys.txt sops private/configs/mosquitto-acl
```

## Testing

Test the connection:

```bash
# Subscribe to test topic
docker exec mosquitto mosquitto_sub -h localhost -u mqtt-admin -P your-password -t test/#

# Publish to test topic (in another terminal)
docker exec mosquitto mosquitto_pub -h localhost -u mqtt-admin -P your-password -t test/message -m "Hello MQTT"
```

## Client Configuration

### Home Assistant

In `configuration.yaml`:

```yaml
mqtt:
  broker: mosquitto
  port: 1883
  username: homeassistant
  password: ha-password
```

### Zigbee2MQTT

Set environment variable in `private/secrets/zigbee2mqtt.env.enc`:

```
ZIGBEE2MQTT_CONFIG_MQTT_SERVER=mqtt://zigbee2mqtt:z2m-password@mosquitto:1883
```

### Z-Wave JS UI

In the Z-Wave JS UI web interface:

- MQTT Server: `mqtt://mosquitto:1883`
- Username: `zwave-js-ui`
- Password: `zwave-password`

## Troubleshooting

Check logs:

```bash
docker logs mosquitto
# or
tail -f /srv/stacks/mosquitto/log/mosquitto.log
```

Verify config files are deployed:

```bash
ls -la /srv/stacks/mosquitto/
cat /srv/stacks/mosquitto/passwd
```

Test authentication:

```bash
docker exec mosquitto mosquitto_sub -h localhost -u mqtt-admin -P wrong-password -t test
# Should fail with authentication error
```
