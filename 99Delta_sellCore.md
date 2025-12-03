```md
 
📝 แพ็ตล่าสุด อัพเดด เยอะสุดคือ โครงสร้าง  และ เพิ่ม Type การ เข้า รหัส มา ประมาณ 12 type แต่ที่ ใช้ กับ bone ,xyz มี เพิ่มมี 3 type หลัก
    และมีเรื่อง offset ที่ ต้อง เพิ่มตามที่ พี่ไล่ เพราะมันกวนเป้าหมายมันเพื่อ หลบ SDK dumper-7 ละมั้ง ถ้า ดูใน sdk ไม่พบพวกนี้ จะลงไปเรื่อยตามลำดับละกัน
   ├─ 1️⃣ ⚙️ชุดโครงสร้างหลัก vector3.h
   ├───────── ① FQuat โม
   ├───────── ② Rotation โม
   ├───────── ③ Vector3 โม
   ├───────── ④ FTransform  ❓ตัวฝั่งเรา0x30  ทีมศัรตุฝั่ง 0x40   ทั้ง 2 มี decode ตามทีม เช็คจาก ทั้งหมด 12 type ทำ ออโต้ แก้ ไป ไม่ได้ดู ผล ว่า มันกี่อันกันแน่    
   ├───────── ⑤ FEncVector ของกล้อง  
   ├───────── ⑥ ฟังชั้น Decode auto แก้พวก FEncVector ,FQuat,FTransform,scale  =ToMatrixWithScale ,RotatorToMatrix,MatrixMultiplication
   ├─ 2️⃣ 💻ชุดโครงสร้างย้อย พี่วางหน้า main.cpp
   ├───────── ① float GetDistanceM(const Vector3& a, const Vector3& b)❓ ใช้คำนวณระยะที่ แม่น สุด จาก offset เพิ่ม
   ├───────── ② โครงสร้าง FCameraCacheEntry มีอัพ FMinimalViewInfo
   ├─ 2️⃣ 🧠ชุดฟังชั้นอ่าน ทั้งแบบ อ่านเร็วและ แบบ เข้าพร้อมแก้การเข้ารหัส
   ├───────── ① std::vector<unsigned char> readmappangMen128ibk(uint64_t AddressStart, size_t sizepang) ❓ใข้อ่านแล้วGet offset เอา จะไม่กิน cpu
   ├───────── ② std::vector<unsigned char> ReadBlockDecoded(uint64_t addrStart, size_t size) ❓ใช้งาน ใช้อ่าน ทีเดียว ส่วน C2W + ถอดรหัสไป ด้วย จุดสำคัญคนอื่นวาดทีม2 ไม่ผ่าน ต้อง call
   ├───────── ③ std::vector<uint8_t> ReadBlock(uint64_t address, size_t size) ❓ แบบ อัน1 แค่ คืนเป็น uint8_t 
   ├───────── ④ uint64_t ReadChainSafe(uint64_t base, const std::vector<uint64_t>& chain) ❓ใช้งานอ่านเผื่อoffset ไม่แน่ใจหลาย offset ใส่ไปได้ แบบ จะอ่าน 0x728,0x738,0x748  ข้อมูล นี้ บางaddress มัน สลับที่offset  ดักทางไว้ จริงๆ อ่าน แค่ 2 อันก็ได้ เผื่อ เฉยๆ วิธีเรียก  uint64_t arr1 = ReadChainSafe(mesh, { 0x728, 0x10, 0x20 });  
   ├─ 3️⃣ 🔧🛠ชุดฟังชั้นตรวจสอบและการเข้ารหัส
   ├─ 4️⃣ 🔥ชุดโค้ดการทำงาน ทั้งส่วนเทรนอัพเดดค่าและเทรนหลักวาด Esp 
 
 
 
```

---
 🔥ชุดโค้ดการทำงาน ทั้งส่วนเทรนอัพเดดค่าและเทรนหลักวาด Esp 
---
```cpp
 
//ฟังชั้นใหม่ เขียนตอนทำ Node พึ่งรู้มันไม่ได้ เขียน ไว้ งี้ 
uintptr_t GetBoneArray(uintptr_t mesh)
{
	uint64_t b1 = ReadChainSafe(mesh, { 0x728, 0x0 , 0x10 });
	uint64_t b2 = ReadChainSafe(mesh, { 0x728, 0x10 , 0x20 });
	if (b1 > 0x1000) return b1;
	if (b2 > 0x1000) return b2;
	return 0;
}
//ทดลองแล้วอาจช้า
uintptr_t GetBoneArray(uintptr_t mesh)
{
	 
	uintptr_t b1  = driver.read<uintptr_t>(mesh + 0x728);
    if (IsNibbleTail(b1) == true) b1  = TrimTrailingNibble(driver.read<uintptr_t>(mesh + 0x728));
	uintptr_t b2  = driver.read<uintptr_t>(mesh + 0x738); 
    if (IsNibbleTail(b2) == true) b2 = TrimTrailingNibble(driver.read<uintptr_t>(mesh + 0x738)); 
	 uintptr_t b3  = driver.read<uintptr_t>(mesh + 0x748); 
    if (IsNibbleTail(b3) == true) b3 = TrimTrailingNibble(driver.read<uintptr_t>(mesh + 0x748));
//	uintptr_t arr = TrimTrailingNibble(ReadChain(mesh, { 0x728 ,0x738,0x748 }));
   
	if (b1 > 0x1000) return b1;
	if (b2 > 0x1000) return b2;
	 if (b3 > 0x1000) return b3; 
	return 0;
}

uintptr_t GetBonePtr30(uintptr_t mesh, int index)
{
	uint64_t arr1 = ReadChainSafe(mesh, { 0x728, 0x10, 0x20 });

	// uintptr_t arr1 = GetBoneArray(mesh);
	  uintptr_t arrde = FixBoneArrayPtr(arr1);
		auto arr =  (arr1 > 0x1000)? arr1: arrde;
 //
	return arr ? arr + index * 0x30 : 0;
}
FTransform GetBoneTransform(uintptr_t mesh, int id)
{ 
	uintptr_t arr = GetBoneArray(mesh);
 
	if (!arr) return FTransform(); 
	// First try normal 0x30 struct
	uintptr_t p30 = arr + id * 0x30;
	FTransform t30 = driver.read<FTransform>(p30);
	 
	if (ValidBoneT(t30))
		return t30; 
 


	uintptr_t arr40 = FixBoneArrayPtr(arr);
	uintptr_t p40= arr40 + id * 0x40;
	uintptr_t realPtrb  = ResolveBone40(p40);

	uintptr_t realPtr = FixBoneArrayPtr(realPtrb);
	if (realPtr > 0x1000)
	{
		FTransform t40 = driver.read<FTransform>(realPtr);
		if (ValidBoneT(t40))
			return t40;
	}

	return FTransform();
}
 
#include <unordered_map>

inline std::unordered_map<uint64_t, int> g_C2WCache;
FTransform TryReadC2W(uintptr_t mesh)
{
	// ----------- 0) ถ้ามี cache → ใช้ offset เดิมทันที -----------
	auto it = g_C2WCache.find(mesh);
	if (it != g_C2WCache.end())
	{
		int off = it->second;
		FTransform t = driver.read<FTransform>(mesh + off);

		if (ValidC2W(t))
			return t;             // ใช้ offset เดิมสำเร็จ

		// ถ้าใช้ไม่ได้ ให้ลบ cache แล้ว scan ใหม่
		g_C2WCache.erase(mesh);
	}

	// ----------- 1) เตรียม block read ครั้งเดียว -----------
	const int START = 0x190;
	const int END = 0x3D0;
	const int SIZE = END - START + sizeof(FTransform);

	auto buf = ReadBlockDecoded(mesh + START, SIZE);

	auto LocalRead = [&](int off) -> FTransform
		{
			int local = off - START;
			return *(FTransform*)(&buf[local]);
		};

	// ----------- 2) candidate offsets แบบเต็มระบบ -----------
	static int cand[] = {
		0x210,0x220,0x230,0x240,0x250,
		0x260,0x270,0x280,0x290,0x2A0,
		0x2B0,0x2C0,0x2D0,0x2E0,0x2F0,
		0x300,0x310,0x320,0x330,0x340,
		0x350,0x360,0x370,0x380,0x390,
		0x3A0,0x3B0,0x3C0,0x3D0
	};

	// ----------- 3) Scan หา offset ที่ valid -----------
	for (int off : cand)
	{
		FTransform t = LocalRead(off);

		if (ValidC2W(t))
		{
			// บันทึก cache
			g_C2WCache[mesh] = off;

			//return ค่า C2W
			return t;
		}
	}

	// ----------- 4) ไม่เจออะไรเลย -> return empty -----------
	return FTransform{};
}

Vector3 GetBoneWithRotation(uintptr_t mesh, int id)
{
	FTransform bone = GetBoneTransform(mesh, id);
	//bone.EncHandler.bDynamic =0;
	
	//driver.write< uint8_t>(bone.EncHandler.bDynamic, 0);
	FTransform c2w = TryReadC2W(mesh);
	//c2w.EncHandler.bDynamic = 0;

	XMMATRIX m = MatrixMultiplication(
		bone.ToMatrixWithScale(),
		c2w.ToMatrixWithScale()
	);

	return { m.r[3].m128_f32[0],
			 m.r[3].m128_f32[1],
			 m.r[3].m128_f32[2] };
}


```
---
🔧🛠 ชุด ฟังชั้นตรวจสอบและการถอดรหัส
---

```cpp

bool isBadPtr(intptr_t ptr)
{
	return false;
	constexpr auto minimum_application_address = intptr_t(0x0001000);
	 
	constexpr auto maximum_application_address = 0x7FFFFFFEFFFF;

	return (ptr < minimum_application_address || ptr > maximum_application_address);
}

uintptr_t TrimTrailingNibbleEp2(uintptr_t addr)
{
	// อ่าน low byte (สอง hex หลักสุดท้าย)
	uint8_t lowByte = static_cast<uint8_t>(addr & 0xFFu);

	// high nibble = upper 4 bits of lowByte
	uint8_t highNibble = (lowByte & 0xF0u) >> 4;
	// low nibble = lower 4 bits of lowByte
	uint8_t lowNibble = (lowByte & 0x0Fu);

	// เงื่อนไข: high nibble ต้องเป็น 0 และ low nibble ต้องอยู่ในช่วง 1..0xF
	if (highNibble == 0x0 && lowNibble >= 1 && lowNibble <= 0xF) {
		// ตัด nibble ท้ายออก (หารด้วย 16) -> เทียบเท่า shift right 4
		return (addr >> 4); // หรือ addr / 0x10
	}

	// กรณีอื่น คืนค่าเดิม
	return addr;
}
 
uintptr_t TrimTrailingNibble(uintptr_t addr)
{
	 //เช็คท้ายสุด 
	addr = TrimTrailingNibbleEp2(addr);

	//-------------------------------------------------------------
	// 2) หา nibble สูงสุดแบบไม่ใช้ลูป (ใช้ BitScanReverse64)
	//-------------------------------------------------------------
	unsigned long msb = 0;
	if (_BitScanReverse64(&msb, addr))
	{
		int msNib = msb >> 2;     // ตำแหน่ง nibble (0..15)

		if (msNib >= 3)
		{
			// อ่าน 4 nibble (16 bits) ด้วย MAKELONG/MAKEWORD
			WORD group16 = (WORD)((addr >> ((msNib - 3) * 4)) & 0xFFFF);

			BYTE nib0 = HIBYTE(group16);        // nibble X อยู่ตรงนี้
			BYTE nib1 = HIBYTE(group16) & 0x0F; // เอา nib1 จาก nibble
			BYTE nib2 = LOBYTE(group16) >> 4;   // nib2
			BYTE nib3 = LOBYTE(group16) & 0xF;  // nib3

			// pattern X000
			if (nib0 != 0 && nib1 == 0 && nib2 == 0 && nib3 == 0)
			{
				uintptr_t mask = ~(uintptr_t(0xFULL) << (msNib * 4));
				addr &= mask; // ตัด X ออก (กลายเป็น 0000)
			}
		}
	}


	return addr;
}
  uintptr_t FixBoneArrayPtr(uintptr_t raw)
{
	if (raw == 0 || raw < 0x1000)
		return 0;

	uintptr_t v = raw;

	// ขั้นตอนที่ 1: ถ้า pointer ยาวเกิน 8 bytes → ให้เริ่มลบด้านล่าง
	for (int i = 0; i < 8; i++)
	{
		uint8_t last = v & 0xFF;
		if (last == 0x00)
		{
			v >>= 8;
			continue;
		}

		// ลบ byte padding ที่เจอบ่อย (01, 55)
		else if (last >= 0x01 || last <= 0x55 )
		{
			v >>= 8;
			continue;
		}
		 

		break;
	}

	// ขั้นตอนที่ 2: ลบ zero nibble ท้าย (กรณีแบบ 0000xxxx)
	while ((v & 0xF) == 0)
		v >>= 4;

 
	return v ;
}




```
--- 
🧠ชุดฟังชั้นอ่าน ทั้งแบบ อ่านเร็วและ แบบ เข้าพร้อมแก้การเข้ารหัส 
---
```cpp

std::vector<unsigned char> readmappangMen128ibk(uint64_t AddressStart, size_t sizepang)
{
	std::vector<unsigned char> memoryData = {};
	memoryData.clear();
	if (AddressStart == 0 || sizepang == 0) {};
	size_t moduleSize = sizepang;
	const size_t readBytes = sizepang;
	memoryData.resize(moduleSize);
	unsigned char buffer[0x2000] = {};
	//ReadProcessMemory128Ex2(AddressStart, &buffer, sizepang);
	driver.readmnew(AddressStart, buffer, static_cast<uint32_t>(moduleSize));
	std::copy(buffer, buffer + moduleSize, memoryData.begin());
	return memoryData; 
}


std::vector<unsigned char> ReadBlockDecoded(uint64_t addrStart, size_t size)
{
	std::vector<unsigned char> out(size);
	if (addrStart == 0 || size == 0) return out;

	// อ่าน block
	std::vector<unsigned char> raw(size);
	driver.readmnew(addrStart, raw.data(), (uint32_t)size);

	// copy กลับ
	out = raw;

	// ===== PASS 1 — decode pointer-like values =====
	for (size_t i = 0; i + 8 <= size; i += 8)
	{
		uintptr_t v = *(uintptr_t*)(&out[i]);

		// เงื่อนไขว่าควร decode:
		if (v > 0x1000 || (v & 0xFF) == 0x55 || (v & 0xF) == 0)
		{
			v = TrimTrailingNibble(v);
			v = TrimTrailingNibbleEp2(v);

			// เขียนกลับ
			*(uintptr_t*)(&out[i]) = v;
		}
	}

	return out;
}

std::vector<uint8_t> ReadBlock(uint64_t address, size_t size)
{
	std::vector<uint8_t> out(size);
	if (address == 0 || size == 0)
		return out;

	driver.readmnew(address, out.data(), (uint32_t)size);
	return out;
}

 
uint64_t ReadChainSafe(uint64_t base, const std::vector<uint64_t>& chain)
{
	uint64_t ptr = base;

	for (auto off : chain)
	{
		// อ่าน pointer
		uint64_t next = driver.read<uint64_t>(ptr + off);

		// fix tail (000055 / 000001)
		next = TrimTrailingNibble(next);

		if (next < 0x1000)
			return 0;

		ptr = next;
	}

	return ptr;
}


```


---
✔️ชุดโครงสร้างย้อย พี่วางหน้า main.cpp
---
```cpp

struct FMinimalViewInfo
{
		Vector3 Location; 
		FEncHandler EncHandler;//Vector3.h
	    FRotator Rotation; 
	   float FOV ; //CameraCachePrivate.POV.FOV  
	   
};
struct FCameraCacheEntry
{
	float Timestamp; // 0x0000(0x0004)
	char pad_0x4[0xC]; // 0x0004(0x000C)
	struct FMinimalViewInfo POV; // 0x0010(0x09E0)

};
FCameraCacheEntry CameraCachePrivate;

float GetDistanceM(const Vector3& a, const Vector3& b)
{
	if (!IsValidLocation(a) || !IsValidLocation(b))
		return 0.f;

	return a.GetDistanceToInMeters(b);
}

```
 
---
⚙️ชุดโครงสร้าง +อันเก่าขี้เกีตรลบเผื่อ จะดีบัค แต่ ดูตามการเรียกใช้งาน เอา นะ 
---

```cpp
//Vector3.h
#pragma once 
 
#include <DirectXMath.h>
#include <cmath> 
#define M_PI 3.14159265358979323846264338327950288419716939937510

using namespace DirectX;

// ===========================================================
//  FRotator — full-feature (UE style + Vector3 behavior)
// ===========================================================
class FRotator
{
public:
	//using UnderlayingType = float;

	float x;  // Pitch
	float y;  // Yaw
	float z;  // Roll
 
	// ---------------- Constructors ----------------
	FRotator() : x(0.f), y(0.f), z(0.f) {}
	FRotator(float _x, float _y, float _z) : x(_x), y(_y), z(_z) {}
	~FRotator() {}
	  
	// ---------------- Angle helpers ----------------
	inline static float ClampAxis(float Angle)
	{
		Angle = std::fmodf(Angle, 360.f);
		if (Angle < 0.f) Angle += 360.f;
		return Angle;
	}

	inline	static float NormalizeAxis(float Angle)
	{
		Angle = ClampAxis(Angle);
		if (Angle > 180.f)
			Angle -= 360.f;
		return Angle;
	}

	inline	FRotator GetNormalized() const
	{
		return FRotator(
			NormalizeAxis(x),
			NormalizeAxis(y),
			NormalizeAxis(z)
		);
	}

	inline	bool IsZero() const
	{
		return ClampAxis(x) == 0.f &&
			ClampAxis(y) == 0.f &&
			ClampAxis(z) == 0.f;
	}

	inline	float Magnitude() const
	{
		return std::sqrt(x * x + y * y + z * z);
	}

	inline	float Distance(const FRotator& o) const
	{
		return (o - *this).Magnitude();
	}

	// ---------------- Operators ----------------
	inline FRotator operator+(const FRotator& o) const
	{
		return { x + o.x, y + o.y, z + o.z };
	}

	inline FRotator operator-(const FRotator& o) const
	{
		return { x - o.x, y - o.y, z - o.z };
	}

	inline FRotator operator*(float s) const
	{
		return { x * s, y * s, z * s };
	}

	inline FRotator operator/(float s) const
	{
		if (s == 0.f) return *this;
		return { x / s, y / s, z / s };
	}

	inline bool operator==(const FRotator& o) const
	{
		return x == o.x && y == o.y && z == o.z;
	}

	inline bool operator!=(const FRotator& o) const
	{
		return !(*this == o);
	}

	inline FRotator& operator+=(const FRotator& o)
	{
		x += o.x; y += o.y; z += o.z;
		return *this;
	}

	inline FRotator& operator-=(const FRotator& o)
	{
		x -= o.x; y -= o.y; z -= o.z;
		return *this;
	}

	inline FRotator& operator*=(float s)
	{
		x *= s; y *= s; z *= s;
		return *this;
	}

	inline FRotator& operator/=(float s)
	{
		if (s != 0.f)
		{
			x /= s; y /= s; z /= s;
		}
		return *this;
	}

	inline FRotator& operator=(const FRotator& o)
	{
		x = o.x;
		y = o.y;
		z = o.z;
		return *this;
	}
};
 
class Vector3
{
public:
	float x, y, z;

	Vector3() : x(0.f), y(0.f), z(0.f) {}
	Vector3(float _x, float _y, float _z) : x(_x), y(_y), z(_z) {}
	~Vector3() {}
	// ---------------- Constructors ----------------
	 
	inline static float ClampAxis(float Angle)
	{
		Angle = std::fmodf(Angle, 360.f);
		if (Angle < 0.f) Angle += 360.f;
		return Angle;
	}

	inline 	static float NormalizeAxis(float Angle)
	{
		Angle = ClampAxis(Angle);
		if (Angle > 180.f)
			Angle -= 360.f;
		return Angle;
	}

	inline Vector3 GetNormalized() const
	{
		return Vector3(
			NormalizeAxis(x),
			NormalizeAxis(y),
			NormalizeAxis(z)
		);
	}

	inline bool IsZero() const
	{
		return ClampAxis(x) == 0.f &&
			ClampAxis(y) == 0.f &&
			ClampAxis(z) == 0.f;
	}

	inline float Magnitude() const
	{
		return std::sqrt(x * x + y * y + z * z);
	}
	// ---------------- Basic Math ----------------
	inline   float Dot(const Vector3& v) const
	{
		return x * v.x + y * v.y + z * v.z;
	}
	 

	  inline   float Length() const { return Magnitude(); }

 
	  inline   float Distance(const Vector3& v) const
	{
		return std::sqrt((v.x - x) * (v.x - x) +
			(v.y - y) * (v.y - y) +
			(v.z - z) * (v.z - z));
	}

	  inline   float GetDistanceTo(const Vector3& v) const
	{
		return (v - *this).Magnitude();
	}

	  inline float GetDistanceToInMeters(const Vector3& v) const
	{
		return GetDistanceTo(v) * 0.01f;
	}

 
	// ---------------- Operators ----------------
	  inline   Vector3 operator+(const Vector3& v) const
	{
		return Vector3(x + v.x, y + v.y, z + v.z);
	}

	  inline   Vector3 operator-(const Vector3& v) const
	{
		return Vector3(x - v.x, y - v.y, z - v.z);
	}

	  inline   Vector3 operator*(float s) const
	{
		return Vector3(x * s, y * s, z * s);
	}

	  inline  Vector3 operator*(const Vector3& v) const
	{
		return Vector3(x * v.x, y * v.y, z * v.z);
	}
	 
	  inline   Vector3 operator/(float s) const
	{
		if (s == 0.f) return *this;
		return Vector3(x / s, y / s, z / s);
	}

	  inline   Vector3 operator/(const Vector3& v) const
	{
		if (v.x == 0.f || v.y == 0.f || v.z == 0.f)
			return *this;
		return Vector3(x / v.x, y / v.y, z / v.z);
	}

	  inline 	  bool operator==(const Vector3& v) const
	{
		return x == v.x && y == v.y && z == v.z;
	}

	  inline   bool operator!=(const Vector3& v) const
	{
		return !(*this == v);
	}

	  inline   Vector3& operator+=(const Vector3& v)
	{
		x += v.x; y += v.y; z += v.z;
		return *this;
	}

	  inline   Vector3& operator-=(const Vector3& v)
	{
		x -= v.x; y -= v.y; z -= v.z;
		return *this;
	}

	  inline 	  Vector3& operator*=(float s)
	{
		x *= s; y *= s; z *= s;
		return *this;
	}

	  inline   Vector3& operator/=(float s)
	{
		if (s != 0.f) { x /= s; y /= s; z /= s; }
		return *this;
	}

	  inline   Vector3& operator=(const Vector3& other)
	{
		x = other.x;
		y = other.y;
		z = other.z;
		return *this;
	}
	 
};


// Vector4 Definition
struct Vector4
{
public:
	float x, y, z, w;
	inline Vector4() : x(0.f), y(0.f), z(0.f), w(0.f) {}
	inline Vector4(float _x, float _y, float _z, float _w) : x(_x), y(_y), z(_z), w(_w) {}
};
 
 
struct FEncHandler
{
	uint16_t  Index;
	int8_t bEncrypted ;
	uint8_t bDynamic;
};

struct FEncVector
{
	float x;
	float y;
	float z;
	FEncHandler EncHandler;
};
 
inline Vector3 Decode(const FEncVector& v)
{
	if (!v.EncHandler.bEncrypted)
		return Vector3(v.x, v.y, v.z);

	// pattern เดียวกับ bone → 0.7071f
	return Vector3(v.x * 0.7071f, v.y * 0.7071f, v.z * 0.7071f);
}
 
// FQuat Definition
struct FQuatdeffff
{
	float x, y, z, w;
};

struct FQuat
{
public:
	float x, y, z, w;
	 
	 FQuat(float X = 0, float Y = 0, float Z = 0, float W = 1) : x(X), y(Y), z(Z), w(W)
	{
	}

	FQuat(const FQuat& q) : x(q.x), y(q.y), z(q.z), w(q.w)
	{
	}
	inline FQuat DecodeQuat(const FEncVector& v) const
	{
		if (!v.EncHandler.bEncrypted)
			return v.x;

		float k = 0.7071f;

		// ถ้าค่า enc เป็นลบ → ต้องกลับด้าน quaternion
		if (v.EncHandler.bEncrypted < 0)
			k = -0.7071f;

		return FQuat(x * k, y * k, z * k, w * k);
	}

	float Magnitude() const
	{
		return sqrtf(x * x + y * y + z * z + w * w);
	}

	inline FQuat GetNormalized() const
	{
		float m = Magnitude();
		if (m < 1e-6f) return *this;
		return FQuat(x / m, y / m, z / m, w / m);
	}

	// quaternion * quaternion
	inline FQuat operator*(const FQuat& o) const
	{
		return FQuat(
			w * o.x + x * o.w + y * o.z - z * o.y,
			w * o.y - x * o.z + y * o.w + z * o.x,
			w * o.z + x * o.y - y * o.x + z * o.w,
			w * o.w - x * o.x - y * o.y - z * o.z
		);
	}

	 
};
struct FTransform
{
	FQuat rot;               // 0x00
	Vector3 translation;     // 0x10
	float unnkow0 ;
	Vector3 scale;           // 0x20  
    float unnkow1 ;
	FEncHandler EncHandler;  // 0x30
	Vector3 unnkow2 ;       // 0x34 – 0x3F 
	inline	FQuat DecodeQuat() const
	{
		// ไม่เข้ารหัส
		if (!EncHandler.bEncrypted)
			return rot;

		// pattern จาก engine (multiple 0.7071)
		return FQuat(rot.x * 0.7071f, rot.y * 0.7071f, rot.z * 0.7071f, rot.w * 0.7071f);
	}

	inline  XMMATRIX ToMatrixWithScale() const
	{
		FQuat q = DecodeQuat().GetNormalized();

		float x2 = q.x + q.x;
		float y2 = q.y + q.y;
		float z2 = q.z + q.z;

		float xx2 = q.x * x2;
		float yy2 = q.y * y2;
		float zz2 = q.z * z2;

		float xy2 = q.x * y2;
		float xz2 = q.x * z2;
		float yz2 = q.y * z2;

		float wx2 = q.w * x2;
		float wy2 = q.w * y2;
		float wz2 = q.w * z2;

		XMMATRIX m;

		m.r[0] = { (1 - (yy2 + zz2)) * scale.x,  (xy2 + wz2) * scale.x,      (xz2 - wy2) * scale.x,      0 };
		m.r[1] = { (xy2 - wz2) * scale.y,       (1 - (xx2 + zz2)) * scale.y, (yz2 + wx2) * scale.y,      0 };
		m.r[2] = { (xz2 + wy2) * scale.z,       (yz2 - wx2) * scale.z,       (1 - (xx2 + yy2)) * scale.z,0 };
		m.r[3] = { translation.x, translation.y, translation.z, 1 };

		return m;
	} 
	inline XMMATRIX ToMatrixWithScaleDef()
	 {
		 XMMATRIX m;
		 m.r[3].m128_f32[0] = translation.x;
		 m.r[3].m128_f32[1] = translation.y;
		 m.r[3].m128_f32[2] = translation.z;

		 float x2 = rot.x + rot.x;
		 float y2 = rot.y + rot.y;
		 float z2 = rot.z + rot.z;

		 float xx2 = rot.x * x2;
		 float yy2 = rot.y * y2;
		 float zz2 = rot.z * z2;
		 m.r[0].m128_f32[0] = (1.0f - (yy2 + zz2)) * scale.x;
		 m.r[1].m128_f32[1] = (1.0f - (xx2 + zz2)) * scale.y;
		 m.r[2].m128_f32[2] = (1.0f - (xx2 + yy2)) * scale.z;

		 float yz2 = rot.y * z2;
		 float wx2 = rot.w * x2;
		 m.r[2].m128_f32[1] = (yz2 - wx2) * scale.z;
		 m.r[1].m128_f32[2] = (yz2 + wx2) * scale.y;

		 float xy2 = rot.x * y2;
		 float wz2 = rot.w * z2;
		 m.r[1].m128_f32[0] = (xy2 - wz2) * scale.y;
		 m.r[0].m128_f32[1] = (xy2 + wz2) * scale.x;

		 float xz2 = rot.x * z2;
		 float wy2 = rot.w * y2;
		 m.r[2].m128_f32[0] = (xz2 + wy2) * scale.z;
		 m.r[0].m128_f32[2] = (xz2 - wy2) * scale.x;

		 m.r[0].m128_f32[3] = 0.0f;
		 m.r[1].m128_f32[3] = 0.0f;
		 m.r[2].m128_f32[3] = 0.0f;
		 m.r[3].m128_f32[3] = 1.0f;

		 return m;
	 }


	inline XMMATRIX RotatorMat(const FRotator& R)
	{
		float Pitch = R.x * (M_PI / 180.f);
		float Yaw = R.y * (M_PI / 180.f);
		float Roll = R.z * (M_PI / 180.f);

		float SP = sinf(Pitch);
		float CP = cosf(Pitch);
		float SY = sinf(Yaw);
		float CY = cosf(Yaw);
		float SR = sinf(Roll);
		float CR = cosf(Roll);

		XMMATRIX m;

		m.r[0] = { CP * CY,             CP * SY,             SP,      0 };
		m.r[1] = { SR * SP * CY - CR * SY, SR * SP * SY + CR * CY, -SR * CP, 0 };
		m.r[2] = { -(CR * SP * CY + SR * SY), CY * SR - CR * SP * SY, CR * CP, 0 };
		m.r[3] = { 0,0,0,1 };

		return m;
	}
	inline XMMATRIX MatrixMultiplication(const XMMATRIX& A, const XMMATRIX& B)
	{
		XMMATRIX O;

		for (int r = 0; r < 4; r++)
		{
			for (int c = 0; c < 4; c++)
			{
				O.r[r].m128_f32[c] =
					A.r[r].m128_f32[0] * B.r[0].m128_f32[c] +
					A.r[r].m128_f32[1] * B.r[1].m128_f32[c] +
					A.r[r].m128_f32[2] * B.r[2].m128_f32[c] +
					A.r[r].m128_f32[3] * B.r[3].m128_f32[c];
			}
		}
		return O;
	}
};
 
inline XMMATRIX RotatorToMatrix(const FRotator& R)
{
	float Pitch = R.x * (M_PI / 180.f);
	float Yaw = R.y * (M_PI / 180.f);
	float Roll = R.z * (M_PI / 180.f);

	float SP = sinf(Pitch);
	float CP = cosf(Pitch);
	float SY = sinf(Yaw);
	float CY = cosf(Yaw);
	float SR = sinf(Roll);
	float CR = cosf(Roll);

	XMMATRIX m;

	m.r[0] = { CP * CY,             CP * SY,             SP,      0 };
	m.r[1] = { SR * SP * CY - CR * SY, SR * SP * SY + CR * CY, -SR * CP, 0 };
	m.r[2] = { -(CR * SP * CY + SR * SY), CY * SR - CR * SP * SY, CR * CP, 0 };
	m.r[3] = { 0,0,0,1 };

	return m;
}

inline XMMATRIX MatrixMultiplication(const XMMATRIX& A, const XMMATRIX& B)
{
	XMMATRIX O;

	for (int r = 0; r < 4; r++)
	{
		for (int c = 0; c < 4; c++)
		{
			O.r[r].m128_f32[c] =
				A.r[r].m128_f32[0] * B.r[0].m128_f32[c] +
				A.r[r].m128_f32[1] * B.r[1].m128_f32[c] +
				A.r[r].m128_f32[2] * B.r[2].m128_f32[c] +
				A.r[r].m128_f32[3] * B.r[3].m128_f32[c];
		}
	}
	return O;
}

 //อาจไม่ได้ใช้อันเก่า
XMMATRIX MatrixMultiplicationeeeee(XMMATRIX pM1, XMMATRIX pM2)
{
	XMMATRIX pOut;
	pOut.r[0].m128_f32[0] = pM1.r[0].m128_f32[0] * pM2.r[0].m128_f32[0] + pM1.r[0].m128_f32[1] * pM2.r[1].m128_f32[0] + pM1.r[0].m128_f32[2] * pM2.r[2].m128_f32[0] + pM1.r[0].m128_f32[3] * pM2.r[3].m128_f32[0];
	pOut.r[0].m128_f32[1] = pM1.r[0].m128_f32[0] * pM2.r[0].m128_f32[1] + pM1.r[0].m128_f32[1] * pM2.r[1].m128_f32[1] + pM1.r[0].m128_f32[2] * pM2.r[2].m128_f32[1] + pM1.r[0].m128_f32[3] * pM2.r[3].m128_f32[1];
	pOut.r[0].m128_f32[2] = pM1.r[0].m128_f32[0] * pM2.r[0].m128_f32[2] + pM1.r[0].m128_f32[1] * pM2.r[1].m128_f32[2] + pM1.r[0].m128_f32[2] * pM2.r[2].m128_f32[2] + pM1.r[0].m128_f32[3] * pM2.r[3].m128_f32[2];
	pOut.r[0].m128_f32[3] = pM1.r[0].m128_f32[0] * pM2.r[0].m128_f32[3] + pM1.r[0].m128_f32[1] * pM2.r[1].m128_f32[3] + pM1.r[0].m128_f32[2] * pM2.r[2].m128_f32[3] + pM1.r[0].m128_f32[3] * pM2.r[3].m128_f32[3];

	pOut.r[1].m128_f32[0] = pM1.r[1].m128_f32[0] * pM2.r[0].m128_f32[0] + pM1.r[1].m128_f32[1] * pM2.r[1].m128_f32[0] + pM1.r[1].m128_f32[2] * pM2.r[2].m128_f32[0] + pM1.r[1].m128_f32[3] * pM2.r[3].m128_f32[0];
	pOut.r[1].m128_f32[1] = pM1.r[1].m128_f32[0] * pM2.r[0].m128_f32[1] + pM1.r[1].m128_f32[1] * pM2.r[1].m128_f32[1] + pM1.r[1].m128_f32[2] * pM2.r[2].m128_f32[1] + pM1.r[1].m128_f32[3] * pM2.r[3].m128_f32[1];
	pOut.r[1].m128_f32[2] = pM1.r[1].m128_f32[0] * pM2.r[0].m128_f32[2] + pM1.r[1].m128_f32[1] * pM2.r[1].m128_f32[2] + pM1.r[1].m128_f32[2] * pM2.r[2].m128_f32[2] + pM1.r[1].m128_f32[3] * pM2.r[3].m128_f32[2];
	pOut.r[1].m128_f32[3] = pM1.r[1].m128_f32[0] * pM2.r[0].m128_f32[3] + pM1.r[1].m128_f32[1] * pM2.r[1].m128_f32[3] + pM1.r[1].m128_f32[2] * pM2.r[2].m128_f32[3] + pM1.r[1].m128_f32[3] * pM2.r[3].m128_f32[3];

	pOut.r[2].m128_f32[0] = pM1.r[2].m128_f32[0] * pM2.r[0].m128_f32[0] + pM1.r[2].m128_f32[1] * pM2.r[1].m128_f32[0] + pM1.r[2].m128_f32[2] * pM2.r[2].m128_f32[0] + pM1.r[2].m128_f32[3] * pM2.r[3].m128_f32[0];
	pOut.r[2].m128_f32[1] = pM1.r[2].m128_f32[0] * pM2.r[0].m128_f32[1] + pM1.r[2].m128_f32[1] * pM2.r[1].m128_f32[1] + pM1.r[2].m128_f32[2] * pM2.r[2].m128_f32[1] + pM1.r[2].m128_f32[3] * pM2.r[3].m128_f32[1];
	pOut.r[2].m128_f32[2] = pM1.r[2].m128_f32[0] * pM2.r[0].m128_f32[2] + pM1.r[2].m128_f32[1] * pM2.r[1].m128_f32[2] + pM1.r[2].m128_f32[2] * pM2.r[2].m128_f32[2] + pM1.r[2].m128_f32[3] * pM2.r[3].m128_f32[2];
	pOut.r[2].m128_f32[3] = pM1.r[2].m128_f32[0] * pM2.r[0].m128_f32[3] + pM1.r[2].m128_f32[1] * pM2.r[1].m128_f32[3] + pM1.r[2].m128_f32[2] * pM2.r[2].m128_f32[3] + pM1.r[2].m128_f32[3] * pM2.r[3].m128_f32[3];

	pOut.r[3].m128_f32[0] = pM1.r[3].m128_f32[0] * pM2.r[0].m128_f32[0] + pM1.r[3].m128_f32[1] * pM2.r[1].m128_f32[0] + pM1.r[3].m128_f32[2] * pM2.r[2].m128_f32[0] + pM1.r[3].m128_f32[3] * pM2.r[3].m128_f32[0];
	pOut.r[3].m128_f32[1] = pM1.r[3].m128_f32[0] * pM2.r[0].m128_f32[1] + pM1.r[3].m128_f32[1] * pM2.r[1].m128_f32[1] + pM1.r[3].m128_f32[2] * pM2.r[2].m128_f32[1] + pM1.r[3].m128_f32[3] * pM2.r[3].m128_f32[1];
	pOut.r[3].m128_f32[2] = pM1.r[3].m128_f32[0] * pM2.r[0].m128_f32[2] + pM1.r[3].m128_f32[1] * pM2.r[1].m128_f32[2] + pM1.r[3].m128_f32[2] * pM2.r[2].m128_f32[2] + pM1.r[3].m128_f32[3] * pM2.r[3].m128_f32[2];
	pOut.r[3].m128_f32[3] = pM1.r[3].m128_f32[0] * pM2.r[0].m128_f32[3] + pM1.r[3].m128_f32[1] * pM2.r[1].m128_f32[3] + pM1.r[3].m128_f32[2] * pM2.r[2].m128_f32[3] + pM1.r[3].m128_f32[3] * pM2.r[3].m128_f32[3];

	return pOut;
}



#define M_PI	3.14159265358979323846264338327950288419716939937510

#define M_PI	3.14159265358979323846264338327950288419716939937510
XMMATRIX ToMatrix(Vector3 Rotation, Vector3 origin = Vector3(0, 0, 0));
XMMATRIX ToMatrix(Vector3 Rotation, Vector3 origin)
{
	float Pitch = (Rotation.x * float(M_PI) / 180.f);
	float Yaw = (Rotation.y * float(M_PI) / 180.f);
	float Roll = (Rotation.z * float(M_PI) / 180.f);

	float SP = sinf(Pitch);
	float CP = cosf(Pitch);
	float SY = sinf(Yaw);
	float CY = cosf(Yaw);
	float SR = sinf(Roll);
	float CR = cosf(Roll);

	XMMATRIX Matrix;
	Matrix.r[0].m128_f32[0] = CP * CY;
	Matrix.r[0].m128_f32[1] = CP * SY;
	Matrix.r[0].m128_f32[2] = SP;
	Matrix.r[0].m128_f32[3] = 0.f;

	Matrix.r[1].m128_f32[0] = SR * SP * CY - CR * SY;
	Matrix.r[1].m128_f32[1] = SR * SP * SY + CR * CY;
	Matrix.r[1].m128_f32[2] = -SR * CP;
	Matrix.r[1].m128_f32[3] = 0.f;

	Matrix.r[2].m128_f32[0] = -(CR * SP * CY + SR * SY);
	Matrix.r[2].m128_f32[1] = CY * SR - CR * SP * SY;
	Matrix.r[2].m128_f32[2] = CR * CP;
	Matrix.r[2].m128_f32[3] = 0.f;

	Matrix.r[3].m128_f32[0] = origin.x;
	Matrix.r[3].m128_f32[1] = origin.y;
	Matrix.r[3].m128_f32[2] = origin.z;
	Matrix.r[3].m128_f32[3] = 1.f;

	return Matrix;
}

XMMATRIX Matrix(FRotator rot, Vector3 origin = Vector3(0, 0, 0))
{
	float radPitch = (rot.x * float(M_PI) / 180.f);
	float radYaw = (rot.y * float(M_PI) / 180.f);
	float radRoll = (rot.z * float(M_PI) / 180.f);

	float SP = sinf(radPitch);
	float CP = cosf(radPitch);
	float SY = sinf(radYaw);
	float CY = cosf(radYaw);
	float SR = sinf(radRoll);
	float CR = cosf(radRoll);

	XMMATRIX matrix;
	matrix.r[0].m128_f32[0] = CP * CY;
	matrix.r[0].m128_f32[1] = CP * SY;
	matrix.r[0].m128_f32[2] = SP;
	matrix.r[0].m128_f32[3] = 0.f;

	matrix.r[1].m128_f32[0] = SR * SP * CY - CR * SY;
	matrix.r[1].m128_f32[1] = SR * SP * SY + CR * CY;
	matrix.r[1].m128_f32[2] = -SR * CP;
	matrix.r[1].m128_f32[3] = 0.f;

	matrix.r[2].m128_f32[0] = -(CR * SP * CY + SR * SY);
	matrix.r[2].m128_f32[1] = CY * SR - CR * SP * SY;
	matrix.r[2].m128_f32[2] = CR * CP;
	matrix.r[2].m128_f32[3] = 0.f;

	matrix.r[3].m128_f32[0] = origin.x;
	matrix.r[3].m128_f32[1] = origin.y;
	matrix.r[3].m128_f32[2] = origin.z;
	matrix.r[3].m128_f32[3] = 1.f;

	return matrix;
}
// Calculate Angle
Vector3 CalcAngle(Vector3 src, Vector3 dst)
{
	Vector3 origin = dst - src;
	float dist = sqrtf(origin.x * origin.x + origin.y * origin.y + origin.z * origin.z);
	Vector3 angles(0.f, 0.f, 0.f);
	angles.x = -asinf(origin.z / dist) * (180.f / M_PI);
	angles.y = atan2f(origin.y, origin.x) * (180.f / M_PI);
	return angles;
}

```
---
```
