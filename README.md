# ⚡ vpn-node-setup

**Xray/Remnawave Node Builder** — боевой тюнинг VPN-ноды под высокую нагрузку:
ядро XanMod + BBRv3, datapath/sysctl-пакеты, zram, fq, атомарный fstab, stack.conf
для связки с [shieldnode](https://github.com/SpofyJet/shield).

![version](https://img.shields.io/badge/version-v6.0.0-blue)
![platform](https://img.shields.io/badge/platform-Debian%2012%2B%20%C2%B7%20Ubuntu%2024.04%2B-orange)
![shell](https://img.shields.io/badge/lang-bash-lightgrey)

## Установка — одна команда

```bash
sudo bash <(curl -fsSL https://raw.githubusercontent.com/SpofyJet/node/main/vpn-node-setup.sh) --optimize
```

Пиннинг на конкретный релиз:

```bash
sudo bash <(curl -fsSL https://raw.githubusercontent.com/SpofyJet/node/v6.0.0/vpn-node-setup.sh) --optimize
```

## Что делает

- **XanMod kernel + BBRv3** (РФ-ноды: Acquire::Timeout/Retries + timeout 900 против DPI-сталлов)
- **Datapath pack**: netdev_budget, tcp_max_tw_buckets, fq (buckets 32768, mq-aware),
  tcp_mem floor по RAM, udp_mem без min-обхода pressure
- **Flow offloading** — OPT-IN (`ENABLE_FLOWTABLE=1` софт / `ENABLE_HW_FLOW_OFFLOAD=1`);
  по умолчанию выключен, чтобы established-потоки не обходили учёт ddos_protect
- **fstab атомарно** (tmp в /etc + mv), noatime-persistence через oneshot-юнит при
  отсутствии записи в fstab
- **GRUB**: скан не-LTS ядер по всем /boot/vmlinuz-* (stock 7.0 не перехватит XanMod 6.18)
- **zram** (ZRAM_ALGO, рекомпрессия idle-страниц), vGPU blacklist, IRQ/io-sched/dirty-tune
- **kdump-detect** (Ubuntu 24.10+/26.04), dracut-aware пересборка initramfs
- **stack.conf** `contract_schema=2` — shared keys для валидации shieldnode v4.x
- **Kill-switch'и честные**: удаляют артефакты прошлых прогонов, не оставляют мусор

## Kill-switch'и (env)

| Переменная | Эффект |
|---|---|
| `ENABLE_FLOWTABLE=1` | включить software flow offloading (opt-in) |
| `ENABLE_HW_FLOW_OFFLOAD=1` | hw-probe flow offload |
| `DISABLE_DATAPATH2=1` | отключить datapath pack 2.0 (+ удалить vpn-fq-tune) |
| `SETUP_NO_ZRAM=1` | не трогать zram |
| `DISABLE_SSD_TUNE=1` | отключить io-sched/SSD семейство |
| `SKIP_VGPU_BLACKLIST=1` | не блокировать qxl/bochs_drm/cirrus |
| `ENABLE_LOW_LATENCY_NIC=1` | GRO flush low-latency (измеримо только <20k сессий) |
| `SETUP_DISABLE_KDUMP=1` | снять crashkernel (opt-in) |
| `SETUP_DISABLE_UNATTENDED=0` | вернуть авто-апдейты Ubuntu |

## Управление

```bash
vpn-node-setup --check      # сравнить версию с github
vpn-node-setup --upgrade    # скачать и запустить последнюю версию
vpn-node-setup --help       # полный список флагов и env
```

## Конверт окружений

x86_64 · KVM/Xen/Hyper-V/VMware/bare-metal · Debian 12/13 · Ubuntu 24.04+/26.04.
LXC/OpenVZ/docker блокируются рано (ШАГ 1).

## Changelog v6.0.0

Полный аудит 2026-09 (18 находок). Ключевое:

- flow offloading: opt-out → **opt-in** (контракт со shieldnode)
- fstab атомарно; apt kernel timeout для РФ; GRUB-скан всех ядер
- `sysctl --system` больше не откатывает 90-shieldnode.conf
- udp_rmem/wmem min убраны; fq buckets 32768 + mq; tcp_mem floor
- datapath2: 96- → 81- (security-overrides shieldnode побеждают)
- stack.conf schema 2, kdump-detect, dracut fallback, ZRAM_ALGO

Полная история изменений — в шапке скрипта `vpn-node-setup.sh`.

---

*Деплой этого релиза: [`deploy-github.sh`](https://github.com/SpofyJet/shield) · classic PAT, одна команда.*
