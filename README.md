#include <iostream>
#include <fstream>
#include <string>
#include <iomanip>
#include <ctime>

using namespace std;

struct Menu {
    int    id;
    string nama;
    string kategori;
    string deskripsi;
    double harga;
    bool   tersedia;
};

struct itemKeranjang {
    Menu   menu;       
    int    qty;
    string suhu;       
    string ukuran;
    double subtotal;
};

struct Customer {
    string nama;
    string noHp;
    string metodePembayaran; 
};

struct Pesanan {
    string      idPesanan;
    Customer    customer;
    itemKeranjang keranjang[20];
    int         jumlahItem;
    double      totalHarga;
    double       diskon;
    double      bayar;
    string      waktu;
    string      status;   
};

const int TOTAL_MENU = 12;
Menu daftarMenu[TOTAL_MENU] = {
    {1,  "Americano",          		"Coffee",   	"Espresso dengan air panas, bold & clean",           				18000, true},
    {2,  "Toffee Nut Latte",   		"Coffee",   	"Espresso dengan saus toffee nut dan susu",                			22000, false},
    {3,  "Caramel Macchiato",  		"Coffee",   	"Vanilla latte dengan karamel drizzle manis",         				30000, true},
    {4,  "Aren Latte",    			"Coffee", 		"Kopi susu dengan gula aren asli indonesia",         				22000, true},
    {5,  "Hazelnut Latte",     		"Coffee", 		"Espresso lembut dengan sirup hazelnut",              				30000, true},
    {6,  "Butterscotch Aren Latte", "Coffee", 		"Susu, espresso, sea salt, gula aren asli dan sirup butterscotch",  22000, true},
    {7,  "Matcha Espresso",    		"Coffee",   	"Espresso, susu, dan high quality matcha powder",                   26000, true},
    {8,  "Matcha Latte",       		"Non Coffee", 	"Matcha Jepang premium dengan susu",                  				28000, true},
    {9,  "Caramel Dutch Choco",     "Non Coffee", 	"Dutch chocolate, sirup hazelnut, dan susu",              			27000, true},
    {10, "Milo Dinosaurus",  		"Non Coffee", 	"Susu Milo dengan taburan bubuk milo",               				23000, true},
    {11, "Thai Tea",          		"Non Coffee", 	"Teh khas Thailand dan susu",                    					19000, false},
    {12, "Avocado Caramel",  		"Non Coffee", 	"Alpukat, susu dan saus karamel",             						28000, true}
};

const string FILE_RIWAYAT = "riwayat_pesanan.txt";
itemKeranjang keranjang[20];
int jumlahKeranjang = 0;
int nomorPesanan   = 1;

void bersihLayar() {
    for (int i = 0; i < 3; i++) cout << "\n";
}

string formatRupiah(double nominal) {
    string hasil = "Rp ";
    string angka = to_string((long long)nominal);
    int panjang  = angka.length();
    int mulai    = panjang % 3;

    for (int i = 0; i < panjang; i++) {
        if (i == mulai && i != 0) hasil += ".";
        else if (i > 0 && (i - mulai) % 3 == 0) hasil += ".";
        hasil += angka[i];
    }
    return hasil;
}

string getWaktuSekarang() {
    time_t now = time(0);
    tm* waktu = localtime(&now);
    char temp[80];
    sprintf(temp, "%02d/%02d/%04d %02d:%02d:%02d",
        (*waktu).tm_mday, 1 + (*waktu).tm_mon, 1900 + (*waktu).tm_year,
        (*waktu).tm_hour, (*waktu).tm_min, (*waktu).tm_sec);
    return string(temp);
}

void cetakBanner() {
    cout << "\n";
    cout << "  +==================================================+\n";
    cout << "  |            E A R T H Y C O F F E E               |\n";
    cout << "  |            Every Cup Tells a Story               |\n";
    cout << "  +==================================================+\n";
}

void cetakGaris(char karakter = '-', int panjang = 54) {
    cout << "  ";
    for (int i = 0; i < panjang; i++) cout << karakter;
    cout << "\n";
}

void cetakJudul(string judul) {
    cetakGaris();
    int spasi = (50 - judul.length()) / 2;
    cout << "  ";
    for (int i = 0; i < spasi; i++) cout << " ";
    cout << judul << "\n";
    cetakGaris();
}

void bubbleSortHarga(Menu arr[], int n, bool ascending = true) {
    for (int i = 0; i < n - 1; i++) {
        for (int j = 0; j < n - i - 1; j++) {
            bool perluTukar = ascending;
               if (arr[j].harga > arr[j + 1].harga){
					swap(arr[j], arr[j + 1]);
}
            if (perluTukar) {
                Menu* p = &arr[j];
                Menu* q = &arr[j+1];
                Menu temp = *p;
                *p = *q;
                *q = temp;
            }
        }
    }
}

void selectionSortNama(Menu arr[], int n) {
    for (int i = 0; i < n - 1; i++) {
        int minIdx = i;
        for (int j = i + 1; j < n; j++) {
            if (arr[j].nama < arr[minIdx].nama)
                minIdx = j;
        }
        if (minIdx != i) {
            Menu temp    = arr[i];
            arr[i]       = arr[minIdx];
            arr[minIdx]  = temp;
        }
    }
}

void sortById(Menu arr[], int n) {
    for (int i = 0; i < n - 1; i++)
        for (int j = 0; j < n - i - 1; j++)
            if (arr[j].id > arr[j+1].id) {
                Menu t = arr[j]; arr[j] = arr[j+1]; arr[j+1] = t;
            }
}

void linearSearch(Menu arr[], int n, string keyword, int hasilIdx[], int* jumlahHasil) {
    *jumlahHasil = 0;
    for (char& c : keyword) c = tolower(c);
    for (int i = 0; i < n; i++) {
        string nama = arr[i].nama;
        for (char& c : nama) c = tolower(c);
        if (nama.find(keyword) != string::npos) {
            hasilIdx[(*jumlahHasil)++] = i;
        }
    }
}

int binarySearchById(Menu arr[], int n, int targetId) {
    int kiri = 0, kanan = n - 1;
    
    while (kiri <= kanan) {
        int mid = (kiri + kanan) / 2;
        
        Menu* m = &arr[mid]; // 
        if ((*m).id == targetId)     
        return mid;
        else if ((*m).id < targetId)
			kiri = mid + 1;
        else 
			kanan = mid - 1;
    }
    return -1;
}

void tampilKatalog(Menu arr[], int n, string judulTampil = "MENU KOPI KENANGAN") {
    cetakJudul(judulTampil);
    string katAktif = "";
    
    for (int i = 0; i < n; i++) {
        Menu* m = &arr[i];
        if ((*m).kategori != katAktif) {
            katAktif = (*m).kategori;
            cout << "\n  >>> " << katAktif << "\n";
            cout << "  " << string(50, '.') << "\n";
        }
        string status = "";
			if (!(*m).tersedia) {
				status = " [HABIS]";
}
        cout << "  [" << setw(2) << right << (*m).id << "] "
             << setw(22) << left  << (*m).nama
             << setw(14) << right << formatRupiah((*m).harga)
             << status << "\n";
        cout << "       -> " << (*m).deskripsi << "\n";
    }
    cout << "\n";
}

void halamanKatalog() {
    Menu salinMenu[TOTAL_MENU];
    for (int i = 0; i < TOTAL_MENU; i++) salinMenu[i] = daftarMenu[i];

    int pilihan = 0;
    cout << "\n";
    cout << "\n  Tampilkan menu dengan urutan:\n";
    cout << "  1. Default (ID)\n";
    cout << "  2. Harga Termahal \n";
    cout << "  3. Harga Termurah \n";
    cout << "  4. Nama A-Z \n";
    cout << "  5. Cari menu...\n";
    cout << "  Pilih [1-5]: ";
    cin  >> pilihan;


    if (pilihan == 1) {
        sortById(salinMenu, TOTAL_MENU);
        tampilKatalog(salinMenu, TOTAL_MENU);

    } else if (pilihan == 2) {
        bubbleSortHarga(salinMenu, TOTAL_MENU, true);
        tampilKatalog(salinMenu, TOTAL_MENU, "MENU - HARGA TERMAHAL");

    } else if (pilihan == 3) {
        bubbleSortHarga(salinMenu, TOTAL_MENU, false);
        tampilKatalog(salinMenu, TOTAL_MENU, "MENU - HARGA TERMURAH");

    } else if (pilihan == 4) {
        selectionSortNama(salinMenu, TOTAL_MENU);
        tampilKatalog(salinMenu, TOTAL_MENU, "MENU - A sampai Z");

    } else if (pilihan == 5) {
        cout << "  Cari menu (ketik nama): ";
        string keyword; cin.ignore(); getline(cin, keyword);

        int hasilIdx[TOTAL_MENU];
        int jumlahHasil = 0;
        linearSearch(salinMenu, TOTAL_MENU, keyword, hasilIdx, &jumlahHasil);

        if (jumlahHasil == 0) {
            cout << "[X] Tidak ada menu yang cocok dengan '" << keyword << "'\n";
        } else {
            cout << "\n  Hasil pencarian '" << keyword << "' (" << jumlahHasil << " menu):\n";
            cetakGaris();
            for (int i = 0; i < jumlahHasil; i++) {
                Menu* m = &salinMenu[hasilIdx[i]];
                
        string status;
			if ((*m).tersedia) {
				status = "[OK] Tersedia";
			} else {
				status = "[X] Habis";
				}
				
			cout << "[" << setw(2) << (*m).id << "]"
                 << setw(22) << left  << (*m).nama
                 << setw(14) << right << formatRupiah((*m).harga)
                 << "  " << status << "\n";
            cout << "       -> " << (*m).deskripsi << "\n";
            }
            cetakGaris();
				}
			} else {
				if(!(cin >> pilihan)) {
                    cin.clear();
                    cin.ignore(1000, '\n');
                    cout << endl;
                    cout << "[Sorry] - Pilihan tidak valid! Silahkan pilih antara menu 1 sampai 5 yaa!" << endl;
                    cout << endl;
                }
			}
		}

void tambahKeKeranjang() {
    if (jumlahKeranjang >= 20) {
        cout << "[X] Keranjang penuh (maks 20 item)!\n";
        return;
    }

    sortById(daftarMenu, TOTAL_MENU);
    tampilKatalog(daftarMenu, TOTAL_MENU);

    cout << "  Masukkan ID menu (0 = batal): ";
    int idMenu; 
    cin >> idMenu;
    if (idMenu == 0) 
    return;

    sortById(daftarMenu, TOTAL_MENU);
    int idx = binarySearchById(daftarMenu, TOTAL_MENU, idMenu);

    if (idx == -1) {
        cout << "[X] ID menu tidak ditemukan!\n";
        return;
    }

    Menu* m = &daftarMenu[idx];
    if (!(*m).tersedia) {
		cout << endl;
        cout << "[X] Maaf, " << (*m).nama << " sedang tidak tersedia.\n";
        cout << endl;
        return;
    }

    itemKeranjang* item = &keranjang[jumlahKeranjang];
    (*item).menu = *m;

    cout << "\n  [Kopi] " << (*m).nama << " - " << formatRupiah((*m).harga) << "\n\n";
	
	int pilSuhu;
	int pilUkuran;
	
    cout << "  Suhu minuman:\n";
    cout << "  1. Hot  (panas)\n";
    cout << "  2. Ice  (dingin)\n";
    cout << "  Pilih [1/2]: ";
    cin >> pilSuhu;
		if (pilSuhu == 1) {
			(*item).suhu = "Hot";
		} else {
			(*item).suhu = "Ice";
	}
	
    double tambahanLarge = 5000;
    cout << "\n  Ukuran:\n";
    cout << "  1. Regular  (" << formatRupiah((*m).harga) << ")\n";
    cout << "  2. Large    (+" << formatRupiah(tambahanLarge) << ")\n";
    cout << "  Pilih [1/2]: ";
    cin >> pilUkuran;
		
		if (pilUkuran == 2) {
			(*item).ukuran = "Large";
		} else {
			(*item).ukuran = "Regular";
		}
		
    double hargaItem;
		if (pilUkuran == 2) {
			hargaItem = (*m).harga + tambahanLarge;
		} else {
			hargaItem = (*m).harga;
		}

    cout << "\n  Jumlah: ";
    cin >> (*item).qty;
    if ((*item).qty <= 0) { cout << "  [X] Jumlah tidak valid!\n"; return; }

   (*item).subtotal = hargaItem * (*item).qty;
    jumlahKeranjang++;

    cout << "\n  [OK] " << (*item).qty << "x " << (*m).nama
         << " (" << (*item).suhu << ", " << (*item).ukuran << ")"
         << " = " << formatRupiah((*item).subtotal) << " ditambahkan!\n"
         << "\n";
}

double hitungTotalRekursif(int index) {
   if(index >= jumlahKeranjang) return 0;

   return keranjang[index].subtotal + hitungTotalRekursif(index + 1);
}

void tampilKeranjang() {
    if (jumlahKeranjang == 0) {
        cout << "\n Keranjang kamu kosong. Yuk pesan dulu!\n";
        cout << "\n";
        return;
    }
    
    cetakJudul("KERANJANG KAMU");
    
    for (int i = 0; i < jumlahKeranjang; i++) {
      
    itemKeranjang* item = keranjang + i;
        cout << "  " << (i + 1) << ". " << setw(20) << left << (*item).menu.nama
             << " | " << (*item).suhu << " | " << (*item).ukuran
             << " | " << (*item).qty << "x"
             << " | " << formatRupiah((*item).subtotal) << "\n";
    }
    
    cetakGaris();
    
    cout << left << setw(15) << "TOTAL " << ": " << formatRupiah(hitungTotalRekursif(0)) << "\n";
    cout << left << setw(15) << "Jumlah item " << ": "<< jumlahKeranjang << "\n\n";
}

void hapusDariKeranjang() {
    tampilKeranjang();
    if (jumlahKeranjang == 0) return;

	int no;
    cout << "Nomor item yang ingin dihapus (0 = batal): ";
    cin >> no;
		if (no == 0 || no > jumlahKeranjang) 
		return;


    for(int i = no - 1; i < jumlahKeranjang - 1; i++){
        *(keranjang + i) = *(keranjang + i + 1);
    }
    
    jumlahKeranjang--;
    cout << "[OK] Item berhasil dihapus dari keranjang.\n";
    cout << "\n";
}

void simpanRiwayat(Pesanan* p) {
    ofstream file(FILE_RIWAYAT, ios::app);
    
    if (!file.is_open()) {
        cout << "  [X] Gagal menyimpan riwayat.\n";
        cout << "\n";
        return;
    }

    file << "======================================\n";
    file << "ID PESANAN  : " << (*p).idPesanan     << "\n";
    file << "WAKTU       : " << (*p).waktu          << "\n";
    file << "NAMA        : " << (*p).customer.nama  << "\n";
    file << "NO HP       : " << (*p).customer.noHp  << "\n";
    file << "PEMBAYARAN  : " << (*p).customer.metodePembayaran << "\n";
    file << "--------------------------------------\n";
    for (int i = 0; i < (*p).jumlahItem; i++) {
        itemKeranjang* item = &((*p).keranjang[i]);
        file << "  " << (i + 1) << ". " << (*item).menu.nama
             << " (" << (*item).suhu << "/" << (*item).ukuran << ")"
             << " x" << (*item).qty
             << " = Rp " << fixed << setprecision(0) << (*item).subtotal << "\n";
		}
    
		file << "--------------------------------------\n";
		file << "TOTAL       : Rp " << fixed << setprecision(0) << (*p).totalHarga << "\n";
    if ((*p).diskon > 0)
        file << "DISKON      : Rp " << fixed << setprecision(0) << (*p).diskon << "\n";
		file << "DIBAYAR     : Rp " << fixed << setprecision(0) << (*p).bayar    << "\n";
		file << "STATUS      : " << (*p).status << "\n";
		file << "======================================\n\n";
		file.close();
}

void lihatRiwayat() {
    ifstream file(FILE_RIWAYAT);
    if (!file.is_open()) {
        cout << "\n  Belum ada riwayat pesanan.\n";
        cout << "\n";
        return;
    }

    cetakJudul("RIWAYAT PESANAN");
    
    string baris;
    while (getline(file, baris)) {
        cout << "  " << baris << "\n";
    }
    file.close();
}

void cetakStruk(Pesanan* p) {
    cout << "\n";
    cout << "  +---------------------------------------------------------+\n";
    cout << "  |		          EARHTY COFFEE	                    |\n";
    cout << "  |                      Struk Pembelian                    |\n";
    cout << "  +---------------------------------------------------------+\n";
    cout << "  | ID Pesanan : " << setw(43) << left << (*p).idPesanan << "|\n";
    cout << "  | Waktu      : " << setw(43) << left << (*p).waktu     << "|\n";
    cout << "  | Nama       : " << setw(43) << left << (*p).customer.nama << "|\n";
    cout << "  +---------------------------------------------------------+\n";
    
    for (int i = 0; i < (*p).jumlahItem; i++) {
        itemKeranjang* item = &((*p)).keranjang[i];
        string namaItem = (*item).menu.nama + " (" + (*item).suhu + "/" + (*item).ukuran + ")";
        cout << "  | " << left  << namaItem
             << " x" << (*item).qty
             << setw(29) << right << formatRupiah((*item).subtotal) << " |\n";
			}
    
		cout << "  +---------------------------------------------------------+\n";
		cout << "  | Total      : " << setw(42) << right << formatRupiah((*p).totalHarga) << " |\n";
		if ((*p).diskon > 0){
			cout << "  | Diskon     : " << setw(42) << right << "-" + formatRupiah((*p).diskon) << " |\n";
			cout << "  | Bayar      : " << setw(42) << right << formatRupiah((*p).bayar) << " |\n";
			}
		if ((*p).customer.metodePembayaran == "Cash") {
        
        double kembali = (*p).bayar - ((*p).totalHarga - (*p).diskon);
			cout << "  | Kembali    : " << setw(42) << right << formatRupiah(kembali) << " |\n";
    }
    
    cout << "  +---------------------------------------------------------+\n";
    cout << "  |                Terima kasih sudah memesan!              |\n";
    cout << "  |             Setiap tegukan penuh kebahagiaan            |\n";
    cout << "  +---------------------------------------------------------+\n\n";
}

void prosesPembayaran() {
    if (jumlahKeranjang == 0) {
        cout << "\n  Keranjang kamu masih kosong!\n";
        return;
    }

    tampilKeranjang();

    Customer cust;
    Customer* pc = &cust;

    cout << "-- Data Pemesanan --\n";
    cout << "Nama kamu     : ";
    cin.ignore(); 
    getline(cin, (*pc).nama);
    cout << "Nomor HP      : ";
    getline(cin, (*pc).noHp);

	int pilBayar;
	cout << "\n";
    cout << "\n  Metode Pembayaran\n";
    cout << "  1. Cash\n";
    cout << "  2. Transfer Bank\n";
    cout << "  3. QRIS\n";
    cout << "  Pilih [1-3]: ";
    cin >> pilBayar;
    
    switch (pilBayar) {
        case 1: 
			(*pc).metodePembayaran = "Cash";          
			break;
        case 2:
			(*pc).metodePembayaran = "Transfer Bank"; 
			break;
        case 3: 
			(*pc).metodePembayaran = "QRIS";          
			break;
        default:
			(*pc).metodePembayaran = "Cash";
    }

    double total  = hitungTotalRekursif(0);
    double diskon = 0;

    if (total >= 50000) {
        diskon = total * 0.10;
        cout << "\n  [!!] Selamat! Kamu dapat diskon 10% (min. Rp 50.000)\n";
        cout << "Diskon: " << formatRupiah(diskon) << "\n";
    }

    double totalBayar = total - diskon;
		cout << "\n  Total yang harus dibayar: " << formatRupiah(totalBayar) << "\n";

    double uangDibayar = 0;
    if ((*pc).metodePembayaran == "Cash") {
        cout << "\n  Uang yang dibayarkan  : Rp ";
        cin >> uangDibayar;
			if (uangDibayar < totalBayar) {
					cout << "[X] Uang tidak cukup!\n";
            return;
			}
    } else {
        uangDibayar = totalBayar;
        cout << "\n  Silakan transfer/scan QRIS ke:\n";
        cout << " * EARTHY COFFEE - BCA 1234 5678 9012\n";
        cout << " * QRIS: [EARHTY COFFEE MERCHANT]\n";
        cout << "\n";
        cout << " Tekan ENTER setelah pembayaran...\n";
        cin.ignore(); 
        cin.get();
    }

    Pesanan pesanan;
    Pesanan* pp = &pesanan;

    char idSementara[20];
    sprintf(idSementara, "Earthy-%04d", nomorPesanan++);
    (*pp).idPesanan  = string(idSementara);
    (*pp).customer   = *pc;
    (*pp).jumlahItem = jumlahKeranjang;
    (*pp).totalHarga = total;
    (*pp).diskon     = diskon;
    (*pp).bayar      = uangDibayar;
    (*pp).waktu      = getWaktuSekarang();
    (*pp).status     = "Selesai";

    for (int i = 0; i < jumlahKeranjang; i++) {
        *((*pp).keranjang + i) = *(keranjang + i);
    }
    
    simpanRiwayat(pp);
    cetakStruk(pp);
    
    jumlahKeranjang = 0;
    cout << "  Pesanan kamu sedang diproses! Tunggu sebentar ya \n\n";
}

int main() {
	int pilihan;
    cetakBanner();
    
    cout << "\n Selamat datang di Earthy Coffee!\n";
    cout << " Kami siap menemani harimu dengan kopi terbaik \n\n";

    do {
        cetakGaris('=');
        cout << "  MENU UTAMA\n";
        cetakGaris('=');
        
        cout << "  1. Lihat Menu & Cari Kopi\n";
        cout << "  2. Tambah ke Keranjang\n";
        cout << "  3. Lihat Keranjang\n";
        cout << "  4. Hapus dari Keranjang\n";
        cout << "  5. Checkout & Bayar\n";
        cout << "  6. Riwayat Pesanan\n";
        cout << "  7. Keluar\n";
        cetakGaris();
        cout << "  Pilihan kamu: ";
        cin >> pilihan;

        switch (pilihan) {
            case 1: 
				halamanKatalog();
				system("pause");
				system("cls");         
				break;
            case 2: 
				tambahKeKeranjang();  
				system("pause");
				system("cls");    
				break;
            case 3: 
				tampilKeranjang();     
				system("pause");
				system("cls");   
				break;
            case 4: 
				hapusDariKeranjang();  
				system("pause");
				system("cls");  
				break;
            case 5: 
				prosesPembayaran();
				system("pause");
				system("cls");       
				break;
            case 6: 
				lihatRiwayat();   
				system("pause");
				system("cls");        
				break;
            case 7:
                cout << "\n  Sampai jumpa! Jangan lupa minum kopi besok \n\n";
                break;
            default:
                cout << "  [X] Pilihan tidak valid.\n";
        }

    } while (pilihan != 7);

    return 0;
}
