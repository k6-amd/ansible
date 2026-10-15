# Ansible

Le playbook principal configure le compte administrateur sur les hôtes de
l'inventaire staging par défaut. Les rôles `ssh` et `bind` ne sont pas activés
dans `main.yml`.

Le fichier chiffré `roles/add_admin/vars/publickeytoto.yml` doit contenir
`vault_admin_password_hash`, un hash SHA-512 crypt (`$6$...`) du mot de passe
administrateur. Le mot de passe clair ne doit pas être stocké dans le dépôt.

## Contrôles avant exécution

Depuis la racine du dépôt :

```sh
ansible-playbook --syntax-check main.yml
ansible-lint main.yml roles/add_admin roles/bind
```

Après installation de la collection requise par le rôle d'administration :

```sh
ansible-galaxy collection install -r requirements.yml
```

Le workflow GitHub Actions installe aussi automatiquement les collections
déclarées dans `requirements.yml` avant d'exécuter le playbook.

Pour le workflow staging, configurez le secret GitHub `ANSIBLE_VAULT_PASSWORD`
avec le mot de passe utilisé pour chiffrer les variables Ansible Vault. Le
workflow l'écrit dans le fichier de mot de passe attendu par `ansible.cfg`
après le checkout, sans l'afficher dans les logs.

## Rôle BIND

Le rôle valide le fichier rendu avec `named-checkconf` avant de remplacer la
configuration active et ne redémarre le service que si le fichier a changé.
Par défaut, BIND écoute et répond uniquement sur localhost. Pour un serveur DNS
accessible au réseau, définir explicitement les variables du rôle dans
l'inventaire, par exemple `bind_listen_ipv4`, `bind_allow_query` et
`bind_allow_recursion`, en limitant ces listes aux réseaux autorisés.
