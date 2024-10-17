+++
tags = ["git"]
Description = "Bir sunucuda birden fazla repo clone'lamak"
date = "2024-10-17T12:03:03+03:00"
menu = "main"
title = "Bir sunucuda birden fazla repo clone'lamak"
summary = "Bir sunucuda birden fazla repo clone'lamak için farklı deploy keyler oluşturmanız ve ssh config'inizi ayarlamanız gereki."

+++


Github'da birden fazla repository'yi aynı sunucuda clone'lamak ve her repo için farklı SSH key'ler kullanmak gerektiğinde, her repo için farklı bir SSH key tanımlayarak bu key'leri yönetmek mümkündür. Bu adımları takip ederek çözebilirsiniz:

### 1. **Her repo için ayrı SSH key oluşturun**

Her repo için bir SSH key çifti oluşturmanız gerekiyor:

```bash
# Örnek: repo1 için SSH key oluşturma
ssh-keygen -t rsa -b 4096 -C "your_email@example.com" -f ~/.ssh/repo1_key
# Örnek: repo2 için SSH key oluşturma
ssh-keygen -t rsa -b 4096 -C "your_email@example.com" -f ~/.ssh/repo2_key
```

Bu komutlar, repo1 ve repo2 için iki farklı SSH key oluşturacaktır. Her key'in private ve public versiyonları `~/.ssh` dizininde saklanacaktır (örneğin, `repo1_key` ve `repo1_key.pub`).

### 2. **SSH Config dosyasını düzenleyin**

SSH'nin her repository için hangi key'i kullanacağını belirlemek için `~/.ssh/config` dosyasını yapılandırmanız gerekiyor. Bu dosyayı açın (veya oluşturun):

```bash
nano ~/.ssh/config
```

Daha sonra aşağıdaki gibi bir yapı oluşturun:

```bash
# repo1 için config
Host github-repo1
    HostName github.com
    User git
    IdentityFile ~/.ssh/repo1_key

# repo2 için config
Host github-repo2
    HostName github.com
    User git
    IdentityFile ~/.ssh/repo2_key
```

### 3. **Repository URL'lerini düzenleyin**

Artık her SSH key için farklı bir `Host` tanımladığınızdan, repository'yi clone ederken custom hostname'i kullanmanız gerekiyor.

Örnek: `repo1` için

```bash
git clone git@github-repo1:username/repo1.git
```

Örnek: `repo2` için

```bash
git clone git@github-repo2:username/repo2.git
```

Bu şekilde, her repository için belirlediğiniz özel SSH key kullanılacaktır.

### 4. **Deploy Key’leri GitHub’a ekleyin**

Oluşturduğunuz her SSH key'in `.pub` uzantılı public key'ini GitHub'a eklemelisiniz.

1. GitHub’da ilgili repository'ye gidin.
2. Settings > Deploy Keys bölümüne gidin.
3. `Add Deploy Key` seçeneğini seçin ve ilgili `.pub` dosyanızın içeriğini buraya yapıştırın.
4. Her repo için bu adımı tekrarlayın.

### Sonuç:
Bu adımlarla her repository için ayrı SSH key yönetimini sağlayabilir ve sunucunuzda bu repository'leri clone'layabilirsiniz.
