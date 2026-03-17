---
term: Memory-Mapped Files
category: kavram
tags:
  - software
  - operating-systems
  - systems-programming
  - I/O
  - performance
summary: >-
  Bir dosyanın içeriğini doğrudan sanal bellek adres alanına eşleyerek, dosya
  okuma/yazma işlemlerini bellek erişimi gibi gerçekleştirmeyi sağlayan işletim
  sistemi mekanizması.
relatedTerms:
  - race-condition
created: '2026-03-17T11:42:34.307Z'
updated: '2026-03-17T11:42:34.307Z'
confidence: learning
source: claude-cli
---
# Memory-Mapped Files

> Bir dosyanın içeriğini doğrudan sanal bellek adres alanına eşleyerek, dosya okuma/yazma işlemlerini bellek erişimi gibi gerçekleştirmeyi sağlayan işletim sistemi mekanizması.

## Türkçe Karşılık

Bellek Eşlemeli Dosyalar

## Açıklama

Memory-mapped files, işletim sisteminin sanal bellek alt sistemini kullanarak bir dosyanın tamamını veya bir bölümünü sürecin (process) adres alanına eşleyen bir I/O yöntemidir. Bu eşleme sayesinde uygulama, geleneksel read()/write() sistem çağrıları yerine doğrudan pointer aritmetiği ile dosya içeriğine erişebilir. İşletim sistemi, erişilen sayfaları talep üzerine (demand paging) diskten belleğe yükler ve değişiklikleri uygun zamanda diske geri yazar. Bu yaklaşım, kullanıcı alanı ile çekirdek alanı arasındaki veri kopyalama maliyetini ortadan kaldırdığı için özellikle büyük dosyalarda önemli performans kazanımları sağlar.

## Örnekler

### Örnek 1: POSIX mmap() ile dosya okuma (C)

mmap() sistem çağrısı ile dosya sanal belleğe eşlenir. Ardından mapped pointer'ı üzerinden dosya içeriğine doğrudan erişilir. read() çağrısına gerek kalmaz.

```
#include <sys/mman.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdio.h>

int main() {
    int fd = open("data.bin", O_RDONLY);
    off_t size = lseek(fd, 0, SEEK_END);

    // Dosyayı belleğe eşle
    char *mapped = mmap(NULL, size, PROT_READ, MAP_PRIVATE, fd, 0);
    close(fd);

    // Artık dosyaya bir dizi gibi erişebilirsin
    printf("İlk byte: 0x%02x\n", mapped[0]);
    printf("Son byte: 0x%02x\n", mapped[size - 1]);

    munmap(mapped, size);
    return 0;
}
```

### Örnek 2: Windows CreateFileMapping (C/Win32)

Windows'ta CreateFileMapping + MapViewOfFile çifti, POSIX mmap() ile aynı işlevi görür.

```
HANDLE hFile = CreateFile(L"data.bin", GENERIC_READ, 0, NULL, OPEN_EXISTING, 0, NULL);
HANDLE hMap = CreateFileMapping(hFile, NULL, PAGE_READONLY, 0, 0, NULL);
char *view = (char*)MapViewOfFile(hMap, FILE_MAP_READ, 0, 0, 0);

// view[i] ile dosyaya erişim

UnmapViewOfFile(view);
CloseHandle(hMap);
CloseHandle(hFile);
```

### Örnek 3: Süreçler arası paylaşımlı bellek (IPC)

MAP_SHARED bayrağı ile birden fazla süreç aynı bellek bölgesini paylaşabilir. Bu, yüksek performanslı süreçler arası iletişim (IPC) için yaygın bir tekniktir.

```
// Süreç A: Paylaşımlı alan oluştur
int fd = shm_open("/shared_data", O_CREAT | O_RDWR, 0666);
ftruncate(fd, 4096);
char *shared = mmap(NULL, 4096, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
sprintf(shared, "Merhaba Süreç B!");

// Süreç B: Aynı alana bağlan
int fd = shm_open("/shared_data", O_RDONLY, 0);
char *shared = mmap(NULL, 4096, PROT_READ, MAP_SHARED, fd, 0);
printf("%s\n", shared);  // "Merhaba Süreç B!"
```

## Sık Yapılan Hatalar

- Eşlenmiş bölgenin boyutunu dosya boyutunun ötesine taşırmak — SIGBUS sinyaline veya tanımsız davranışa yol açar.
- MAP_SHARED ile yazma yaparken birden fazla süreç arasında senkronizasyon (mutex, semaphore) kullanmamak — race condition oluşur.
- munmap() veya UnmapViewOfFile() çağrısını unutmak — bellek sızıntısına ve dosya kilidinin serbest bırakılmamasına neden olur.
- Çok küçük dosyalar için mmap kullanmak — sayfa hizalama overhead'i nedeniyle read()/write()'tan daha yavaş olabilir.
- Eşlenmiş bellek üzerinde yapılan değişikliklerin anında diske yazıldığını varsaymak — msync() veya FlushViewOfFile() çağrılmadıkça işletim sistemi yazma zamanlamasını kendi belirler.

## İlişkili Terimler

- race-condition

## Kaynaklar

- Advanced Programming in the UNIX Environment, W. Richard Stevens & Stephen A. Rago, 3rd Edition
- Windows System Programming, Johnson M. Hart, 4th Edition
- Linux man pages: mmap(2), msync(2), munmap(2)
