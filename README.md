# Test
Testt
import kotlinx.coroutines.*
import kotlin.random.Random

// 1. GENİŞLETME FONKSİYONU: String sınıfına yeni bir özellik ekliyoruz
fun String.log(message: String) {
    println("[${this}] $message")
}

// 2. SUSPEND FONKSİYONU: Eşyordam içinde çalışabilen, asenkron bir görev
suspend fun fetchUserData(userId: Int): String {
    // Bu kodun asenkron olduğunu belirtir ve mevcut iş parçacığını (thread) bloklamaz
    delay(Random.nextLong(500, 2000)) // 0.5 ile 2 saniye arasında bekleme simülasyonu

    if (userId % 3 == 0) {
        throw Exception("Kullanıcı $userId için sunucu hatası!")
    }

    return "Kullanıcı Verisi (ID: $userId, Ad: Alice)"
}

// 3. ANA FONKSİYON: Programın başlangıç noktası
fun main() = runBlocking {
    val tag = "MAIN_APP"
    tag.log("Uygulama başlatılıyor...")

    // Eşyordamları başlatmak için bir kapsam (Scope) oluşturulur
    val job = CoroutineScope(Dispatchers.IO).launch {
        tag.log("Asenkron görev başlatıldı. UI donmayacak.")
        
        try {
            // Farklı kullanıcılar için verileri paralel olarak çekiyoruz
            val results = (1..5).map { userId ->
                async {
                    // Bu kısım, her kullanıcı için eş zamanlı çalışır
                    "CORO-$userId".log("Veri çekiliyor...")
                    fetchUserData(userId) 
                }
            }.awaitAll() // Tüm asenkron görevlerin bitmesini bekler

            tag.log("Tüm başarılı veriler çekildi:")
            results.filterNotNull().forEach { tag.log(" -> $it") }

        } catch (e: Exception) {
            // Hata fırlatan kullanıcıyı yakalarız (userId % 3 == 0)
            tag.log("Hata yakalandı: ${e.message}")
        }
    }

    // Main thread'de işimize devam edebiliriz (UI bu sırada aktif kalır)
    tag.log("Başlangıçta UI işlemleri yapılıyor (örneğin butonlar)...")

    // Asenkron görevin bitmesini bekle
    job.join()
    
    tag.log("Uygulama sonlandı.")
}
