# Laporan Penjelasan dan Penyelesaian Praktikum Sistem Operasi 2020
# Modul 4
## Kelompok D04
1. Michael Ricky (05111840000078)
2. Yaniar Pradityas Effendi (05111840000047)
# File
Link ke file yang dibuat:
* [ssfs.c](https://github.com/rddmoon/SoalShiftSISOP20_modul4_D04/blob/master/ssfs.c)
## Soal
Berikut adalah detail filesystem rancangan jasir:
1. Enkripsi versi 1:

Jika sebuah direktori dibuat dengan awalan “encv1_”, maka direktori tersebut akan menjadi direktori terenkripsi menggunakan metode enkripsi v1.
Jika sebuah direktori di-rename dengan awalan “encv1_”, maka direktori tersebut akan menjadi direktori terenkripsi menggunakan metode enkripsi v1.
Apabila sebuah direktori terenkripsi di-rename menjadi tidak terenkripsi, maka isi adirektori tersebut akan terdekrip.
Setiap pembuatan direktori terenkripsi baru (mkdir ataupun rename) akan tercatat ke sebuah database/log berupa file.
Semua file yang berada dalam direktori ter enkripsi menggunakan caesar cipher dengan key.
```9(ku@AW1[Lmvgax6q`5Y2Ry?+sF!^HKQiBXCUSe&0M.b%rI'7d)o4~VfZ*{#:}ETt$3J-zpc]lnh8,GwP_ND|jO```

Misal kan ada file bernama “kelincilucu.jpg” dalam directory FOTO_PENTING, dan key yang dipakai adalah 10
“encv1_rahasia/FOTO_PENTING/kelincilucu.jpg” => “encv1_rahasia/ULlL@u]AlZA(/g7D.|_.Da_a.jpg
Note : Dalam penamaan file ‘/’ diabaikan, dan ekstensi tidak perlu di encrypt.
Metode enkripsi pada suatu direktori juga berlaku kedalam direktori lainnya yang ada didalamnya.

2. Enkripsi versi 2:

Jika sebuah direktori dibuat dengan awalan “encv2_”, maka direktori tersebut akan menjadi direktori terenkripsi menggunakan metode enkripsi v2.
Jika sebuah direktori di-rename dengan awalan “encv2_”, maka direktori tersebut akan menjadi direktori terenkripsi menggunakan metode enkripsi v2.
Apabila sebuah direktori terenkripsi di-rename menjadi tidak terenkripsi, maka isi direktori tersebut akan terdekrip.
Setiap pembuatan direktori terenkripsi baru (mkdir ataupun rename) akan tercatat ke sebuah database/log berupa file.
Pada enkripsi v2, file-file pada direktori asli akan menjadi bagian-bagian kecil sebesar 1024 bytes dan menjadi normal ketika diakses melalui filesystem rancangan jasir. Sebagai contoh, file File_Contoh.txt berukuran 5 kB pada direktori asli akan menjadi 5 file kecil yakni: File_Contoh.txt.000, File_Contoh.txt.001, File_Contoh.txt.002, File_Contoh.txt.003, dan File_Contoh.txt.004.
Metode enkripsi pada suatu direktori juga berlaku kedalam direktori lain yang ada didalam direktori tersebut (rekursif).

3. Sinkronisasi direktori otomatis:

Tanpa mengurangi keumuman, misalkan suatu directory bernama dir akan tersinkronisasi dengan directory yang memiliki nama yang sama dengan awalan sync_ yaitu sync_dir. Persyaratan untuk sinkronisasi yaitu:
Kedua directory memiliki parent directory yang sama.
Kedua directory kosong atau memiliki isi yang sama. Dua directory dapat dikatakan memiliki isi yang sama jika memenuhi:
Nama dari setiap berkas di dalamnya sama.
Modified time dari setiap berkas di dalamnya tidak berselisih lebih dari 0.1 detik.
Sinkronisasi dilakukan ke seluruh isi dari kedua directory tersebut, tidak hanya di satu child directory saja.
Sinkronisasi mencakup pembuatan berkas/directory, penghapusan berkas/directory, dan pengubahan berkas/directory.

Jika persyaratan di atas terlanggar, maka kedua directory tersebut tidak akan tersinkronisasi lagi.
Implementasi dilarang menggunakan symbolic links dan thread.

4. Log system:

Sebuah berkas nantinya akan terbentuk bernama "fs.log" di direktori *home* pengguna (/home/[user]/fs.log) yang berguna menyimpan daftar perintah system call yang telah dijalankan.
Agar nantinya pencatatan lebih rapi dan terstruktur, log akan dibagi menjadi beberapa level yaitu INFO dan WARNING.
Untuk log level WARNING, merupakan pencatatan log untuk syscall rmdir dan unlink.
Sisanya, akan dicatat dengan level INFO.
Format untuk logging yaitu:
```[LEVEL]::[yy][mm][dd]-[HH]:[MM]:[SS]::[CMD]::[DESC ...]```

LEVEL    : Level logging
yy       : Tahun dua digit
mm       : Bulan dua digit
dd       : Hari dua digit
HH       : Jam dua digit
MM       : Menit dua digit
SS       : Detik dua digit
CMD      : System call yang terpanggil
DESC     : Deskripsi tambahan (bisa lebih dari satu, dipisahkan dengan ::)

## Pembahasan
```
static const char *dirpath = "/home/yaniarpe/Documents";
char code[] = "9(ku@AW1[Lmvgax6q`5Y2Ry?+sF!^HKQiBXCUSe&0M.b%rI'7d)o4~VfZ*{#:}ETt$3J-zpc]lnh8,GwP_ND|jO";
const int key = 13;
```
Deklarasi untuk direktori, code untuk encrypt_v1, dan key yang ingin dipakai yaitu 13.
```
char *getExt (char *str) {
    char *ext = strrchr (str, '.');
    if (ext == NULL)
        ext = "";
    return ext;
}
```
Fungsi untuk mencari tahu extension dari sebuah file.
```
void encrypt_v1(char *enc){
	if(strcmp(enc,".") == 0 || strcmp(enc,"..") == 0) return;
	int len = strlen(getExt(enc));
	for ( int i = 0; i < strlen(enc)-len ;i++) {
		if(enc[i] != '/'){
			for (int j = 0; j < strlen(code); j++) {
	     		if(enc[i] == code[j]) {
	        		enc[i] = code[(j+key) % strlen(code)];
	        		break;
        	}
			}
		}
	}
}
```
Fungsi encrypt_v1 untuk enkripsi versi 1. Melakukan enkripsi hanya pada nama file.
```
void decrypt_v1(char *dec){
	if(strcmp(dec,".") == 0 || strcmp(dec,"..") == 0) return;
	int f1 = 0;
	int f2 = 0;
	if(strncmp(dec,"encv1_",6)==0){
		f1 = 1;
		f2 = 1;
	}
	int len = strlen(getExt(dec));
	for ( int i = 0; i < strlen(dec)-len; i++) {
		if(dec[i] == '/'){
			f1=0;
		}
		if((f1 != 1 && dec[i] != '/') && f2 == 1){
			for (int j = 0; j < strlen(code); j++) {
	     		if(dec[i] == code[j]) {
	        		dec[i] = code[(j+strlen(code)-key) % strlen(code)];
	        		break;
        		}
			}
		}
	}
}
```
Fungsi decrypt_v1 untuk melakukan decrypt dari enkripsi versi 1 dengan key 13.
```
void logs (char* cmd, char* deskripsi){
    char level[20],name[1000];
    time_t t = time(NULL);
    struct tm tm = *localtime(&t);
    FILE *file = fopen("/home/yaniarpe/fs.log","a");

    if(strcmp(cmd,"RMDIR") == 0 || strcmp(cmd,"UNLINK") == 0){
      strcpy(level,"WARNING");
    }
    else{
      strcpy(level,"INFO");
    }
    int year = tm.tm_year+1900-2000;
    sprintf(name, "%s::%02d%02d%02d-%02d:%02d:%02d::%s::%s", level, year, tm.tm_mon + 1,
    tm.tm_mday,tm.tm_hour, tm.tm_min, tm.tm_sec, cmd, deskripsi);
    fprintf(file,"%s",name);
    fprintf(file,"\n");
    fclose(file);
}
```
Fungsi logs untuk menyimpan daftar command system call yang dilakukan. Command akan disimpan di file fs.log dan setiap command baru yang dilakukan di-append ke file fs.log tersebut.

Kemudian menambahkan operasi-operasi yang dapat dilakukan di Filesystem. Pada file [ssfs.c](https://github.com/rddmoon/SoalShiftSISOP20_modul4_D04/blob/master/ssfs.c) ada getattr, readdir, mkdir, rmdir, rename, read, write, dan unlink.

```
int main(int argc, char *argv[]){
	umask(0);
	return fuse_main(argc, argv, &xmp_oper, NULL);
}
```
Bagian tersebut merupakan main
