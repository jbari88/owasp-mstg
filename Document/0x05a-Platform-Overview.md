## 안드로이드 플랫폼 개요 Android Platform Overview

이 장에서는 설계 관점에서 안드로이드 플랫폼을 소개합니다. 아래 다섯 가지 핵심 영역에 대해 논의하려고 합니다.: This section introduces the Android platform from an architecture point of view. The following five key areas are discussed:

1. 안드로이드 보안 설계 Android security architecture
2. 안드로이드 애플리케이션 구조 Android application structure
3. 내부 프로세스 간 통신 Inter-process Communication (IPC)
4. 안드로이드 애플리케이션 배포 Android application publishing
5. 안드로이드 애플리케이션 공격 포인트 (?) Android application attack surface

안드로이드 플랫폼에 관해 더 상세한 정보를 원한다면 공식 [안드로이드 개발자 문서 웹사이트](https://developer.android.com/index.html "Android Developer Guide")를 방문해보세요. Visit the official [Android developer documentation website](https://developer.android.com/index.html "Android Developer Guide") for more details about the Android platform.

### 안드로이드 보안 설계 Android Security Architecture

안드로이드는 구글이 개발한 리눅스 기반 오픈 소스 플렛폼으로 모바일 OS로 사용됩니다. 오늘날 이 플랫폼은 휴대폰, 태블릿, 웨어러블 기기, TV, 그리고 다른 "스마트" 기기와 같은 다양한 현대 기술의 기반으로 사용되고 있습니다. 전형적인 안드로이드 빌드는 미리 설치된("선탑재") 애플리케이션과 함께 배포되고 구글 플레이 스토어 또는 다른 마켓플레이스에서 받을 수 있는 앱을 설치할 수 있도록 지원하고 있습니다. Android is a Linux-based open source platform developed by Google, which serves as a mobile operating system (OS). Today the platform is the foundation for a wide variety of modern technology, such as mobile phones, tablets, wearable tech, TVs, and other "smart" devices. Typical Android builds ship with a range of pre-installed ("stock") apps and support installation of third-party apps through the Google Play store and other marketplaces.

안드로이드 소프트웨어 스택은 몇가지 레이어로 구성되어 있습니다. 각각의 레이어는 인터페이스를 정의하고 고유의 서비스를 제공합니다. Android's software stack is composed of several different layers. Each layer defines interfaces and offers specific services.

<img src="Images/Chapters/0x05a/android_software_stack.png" alt="Android Software Stack" width="400">

가장 낮은 단계에서 안드로이드는 변형된 리눅스 커널 위에서 동작합니다. 커널 위에는 내장된 하드웨어 컴포넌트와 상호작용할 수 있는 표준 인터페이스인 하드웨어 추상화 레이어(HAL)가 있습니다. HAL 인터페이스 구현체는 공유 라이드러리 모듈에 패키징되어 안드로이드 시스템이 필요할 때 호출할 수 있습니다. 이런 방식으로 애플리케이션은 단말기 하드웨어를 사용할 수 있습니다.- 가령 선탑재 휴대폰 애플리케이션이 디바이스의 마이크와 스피커를 사용할 수 있는 것처럼요.  At the lowest level, Android is based on a variation of the Linux Kernel. On top of the kernel, the Hardware Abstraction Layer (HAL) defines a standard interface for interacting with built-in hardware components. Several HAL implementations are packaged into shared library modules that the Android system calls when required. This is the basis for allowing applications to interact with the device's hardware—for example, it allows a stock phone application to use a device's microphone and speaker.

안드로이드 앱은 대개 Java로 만들어지고 자바 바이트코드와는 약간 차이가 있는 달빅 바이트코드로 컴파일됩니다. 자바 코드를 .class 파일로 컴파일하고 JVM 바이트코드를 `dx` 툴을 사용해 달빅 .dex 포맷 파일로 변환해서 달빅 바이트코드를 생성합니다. Android apps are usually written in Java and compiled to Dalvik bytecode, which is somewhat different from the traditional Java bytecode. Dalvik bytecode is created by first compiling the Java code to .class files, then converting the JVM bytecode to the Dalvik .dex format with the `dx` tool.

<img src="Images/Chapters/0x05a/java_vs_dalvik.png" alt="Java vs Dalvik" width="350">

최신 안드로이드 버전은 이 바이트코드를 안드로이드 런타임(ART) 위에서 동작시킵니다. ART는 안드로이드 기존 런타임이었던 달빅 가상 머신의 후계자입니다. 달빅과 ART는 바이트코드가 실행되는 방식에서 가장 중요한 차이를 보입니다.The current version of Android executes this bytecode on the Android runtime (ART). ART is the successor to Android's original runtime, the Dalvik Virtual Machine. The key difference between Dalvik and ART is the way the bytecode is executed.

달빅에서, 실행할 때 바이트코드가 기계언어로 변환되는데 이를 *just-in-time* (JIT) 컴파일이라고 부릅니다. JIT 컴파일은 성능 측면에서는 좋지 않습니다.: 앱이 실행될 때마다 컴파일이 이루어지기 때문입니다. 성능을 향상시키기 위해서 ART는 *ahead-of-time*(AOT) 컴파일 방식을 도입했습니다. 이름을 통해 알 수 있듯이 앱은 가장 처음 실행되기 전에 컴파일됩니다. 이후 실행할 때마다 미리 컴파일된 기계언어를 사용됩니다. AOT는 전력 소비를 줄이면서 성능을 2배 향상시킵니다.    In Dalvik, bytecode is translated into machine code at execution time, a process known as *just-in-time* (JIT) compilation. JIT compilation adversely affects performance: the compilation must be performed every time the app is executed. To improve performance, ART introduced *ahead-of-time* (AOT) compilation. As the name implies, apps are precompiled before they are executed for the first time. This precompiled machine code is used for all subsequent executions. AOT improves performance by a factor of two while reducing power consumption.

안드로이드 앱은 하드웨어 자원에 직접 접근할 수 없고 샌드박스 위에서 동작합니다. 이러한 방식은 명확하게 리소스와 앱을 통제할 수 있도록 합니다.: 가령 크래싱된 앱은 같은 단말기 위에서 동작하는 다른 앱에 영향을 줄 수 없습니다. 동시에, 안드로이드 런타임은 앱에 할당되는 시스템 자원의 최대 수를 통제할 수 있어서 특정 앱이 너무 많은 자원을 독점할 수 없도록 방지할 수 있습니다. Android apps don't have direct access to hardware resources, and each app runs in its own sandbox. This allows precise control over resources and apps: for instance, a crashing app doesn't affect other apps running on the device. At the same time, the Android runtime controls the maximum number of system resources allocated to apps, preventing any one app from monopolizing too many resources.

#### 안드로이드 사용자와 그룹 Android Users and Groups

안드로이드 운영 시스템이 리눅스에 기반하지만 유닉스 계열의 시스템과 동일한 방식으로 사용자 계정을 실행하지 않습니다. 안드로이드의 경우, 다중 사용자로 리눅스 커널이 앱을 샌드박스 형태로 동작할 수 있도록 지원합니다.: 몇 가지 예외 상황을 제외하고는 각 앱은 분리된 리눅스 사용자 아래서 다른 앱 또는 다른 운영 시스템에 완벽하게 독립된 환경에서 동작됩니다.  Even though the Android operating system is based on Linux, it doesn't implement user accounts in the same way other Unix-like systems do. In Android, the multi-user support of the Linux kernel to sandbox apps: with a few exceptions, each app runs as though under a separate Linux user, effectively isolated from other apps and the rest of the operating system.

[system/core/include/private/android_filesystem_config.h](http://androidxref.com/7.1.1_r6/xref/system/core/include/private/android_filesystem_config.h "android_filesystem_config.h") 파일에서 시스템 프로세스가 미리 정의한 사용자와 그룹 목록을 확인할 수 있습니다. 그 외 앱의 UID(userId)는 설치된 이후에 추가됩니다. 더 자세한 내용은 안드로이드 샌드박스에 대해 기술한 Bin Chen의 [블로그 포스트](https://pierrchen.blogspot.mk/2016/09/an-walk-through-of-android-uidgid-based.html "Bin Chen - AProgrammer Blog - Android Security: An Overview Of Application Sandbox")를 확인하기 바랍니다. The file [system/core/include/private/android_filesystem_config.h](http://androidxref.com/7.1.1_r6/xref/system/core/include/private/android_filesystem_config.h "android_filesystem_config.h") includes a list of the predefined users and groups system processes are assigned to. UIDs (userIDs) for other applications are added as the latter are installed. For more details, check out Bin Chen's [blog post](https://pierrchen.blogspot.mk/2016/09/an-walk-through-of-android-uidgid-based.html "Bin Chen - AProgrammer Blog - Android Security: An Overview Of Application Sandbox") on Android sandboxing.

예를 들어, 안드로이드 7.0 (API 레벨 24)는 아래 시스템 사용자를 정의하고 있습니다. For example, Android 7.0 (API level 24) defines the following system users:

```c
    #define AID_ROOT             0  /* traditional unix root user */
    #define AID_SYSTEM        1000  /* system server */
    #...
    #define AID_SHELL         2000  /* adb and debug shell user */
    #...
    #define AID_APP          10000  /* first app user */
    ...
```

<br/>
<br/>

#### 안드로이드 단말기 암호화 Android Device Encryption

안드로이드에서는 안드로이드 2.3.4(API 레벨 10)부터 단말기 암호화를 지원하고 있으며 큰 변화를 겪어왔습니다. 구글은 안드로이드 6.0(API 레벨 23) 이상을 지원하는 모든 단말기는 스토리지 암호화를 지원하도록 강제하고 있습니다. 일부 저가형 단말기는 성능에 영향을 미칠 수 있기 때문에 강제하고 있지 않습니다. 다음 장에서 단말기 암호화와 알고리즘에 대한 내용을 다룰 것입니다. Android supports device encryption from Android 2.3.4 (API level 10) and it has undergone some big changes since then. Google imposed that all devices running Android 6.0 (API level 23) or higher had to support storage encryption. Although some low-end devices were exempt because it would significantly impact performance. In the following sections you can find information about device encryption and its algorithms.

##### 전체 디스크 암호화 Full-Disk Encryption

안드로이드 5.0(API 레벨 21) 이상은 전체 디스크 암호화를 지원합니다. 이 암호화는 사용자 데이터 영역을 암/복호화할 때 단말기 비밀번호로 보호되는 단 하나의 키를 사용합니다. 이런 종류의 암호화는 더 이상 사용되지 않는 것을 권고하고 있고 가능한 파일 기반 암호화를 사용해야 합니다. 전체 디스크 암호화는 재시작 이후에 사용자가 비밀번호를 입력하지 않으면 전화를 받을 수도 없고 알람이 울리지 않는 단점이 있습니다. Android 5.0 (API level 21) and above support full-disk encryption. This encryption uses a single key protected by the users' device password to encrypt and decrypt the userdata partition. This kind of encryption is now considered deprecated and file-based encryption should be used whenever possible. Full-disk encryption has drawbacks, such as not being able to receive calls or not having operative alarms after a reboot if the user does not enter his password.

##### 파일 기반 암호화 File-Based Encryption

안드로이드 7.0 (API 레벨 24)는 파일 기반 암호화를 지원합니다. 파일 기반 암호화는 다른 키를 이용해 서로 다른 파일을 암호화하기 때문에 독립적으로 복호화할 수 있습니다. 이 암호화를 지원하는 단말기는 직접 부팅 모드 또한 지원합니다. 직접 부팅 모드는 사용자가 비밀번호를 입력하지 않더라도 알람이나 서비스에 접근 가능하도록 해줍니다. Android 7.0 (API level 24) supports file-based encryption. File-based encryption allows different files to be encrypted with different keys so they can be deciphered independently. Devices which support this type of encryption support Direct Boot as well. Direct Boot enables the device to have access to features such as alarms or accessibility services even if the user does not enter his password.

##### 디스크 암호화를 위한 알고리즘 Adiantum

AES는 스토리지 암호화를 위해 최신 안드로이드 단말기에서 사용하고 있습니다. 사실, AES는 가장 널리 사용되는 알고리즘으로, 최신 프로세서 구현에는 암호화 가속 기능이 있는 ARMv8 또는 AES-NI, 확장 기능이 있는 x86과 같은 하드웨어 가속 암호화 및 암호 해독 작업을 제공하는 전용 명령 세트가 있습니다. 하지만 모든 단말기가 발빠르게 스토리지 암호화에 AES를 사용할 수 있는 것은 아닙니다. 특히 안드로이드 고를 실행시키는 저가형 단말기가 그렇죠. 이러한 장치는 일반적으로 하드웨어 가속 AES가 없는 ARM Cortex-A7과 같은 저가형 프로세서를 사용합니다.  AES is used on most modern Android devices for storage encryption. Actually, AES has become such a widely used algorithm that the most recent processor implementations have a dedicated set of instructions to provide hardware accelerated encryption and decryption operations, such as ARMv8 with its Cryptography Extensions or x86 with AES-NI extension.
However, not all devices are capable of using AES for storage encryption in a timely fashion. Especially low-end devices running Android Go. These devices usually use low-end processors, such as the ARM Cortex-A7 which don't have hardware accelerated AES.

Adiantum은 구글의 Paul Crowley와 Eric Biggers가 최소 50 MiB/s에서만 실행할 수 있는 AES를 지원하지 않는 단말기를 위해 디자인한 암호화 구조입니다. Adiantum은 덧셈, 회전, XOR에만 의존합니다; 이러한 연산은 모든 프로세서가 기본적으로 지원하고 있습니다. 따라서 저가형 프로세서도 AES를 사용하는 것보다 4배 빠르게 암호화하고 5배 빠르게 복호화할 수 있습니다.
Adiantum is a cipher construction designed by Paul Crowley and Eric Biggers at Google to fill the gap for that set of devices which are not able to run AES at least at 50 MiB/s. Adiantum relies only on additions, rotations and XORs; these operations are natively supported on all processors. Therefore, the low-end processors can encrypt 4 times faster and decrypt 5 times faster than they would if they were using AES.

Adiantum은 여러가지 암호화 방식의 조합입니다:
Adiantum is a composition of other ciphers:

- NH : 해싱 함수 NH: A hashing function.
- Poly1305: 메세지 인증코드 A message authentication code (MAC).
- XChaCha12: 스트림 암호 A stream cipher.
- AES-256: A single invocation of AES.

Adiantum은 새로운 암호 방식이지만, ChaCha12와 AES-256이 안전한 이상 안전한 알고리즘입니다. Adiantum은 완전히 새로운 암호화 방식이 아니라 잘 알려진 기존 암호화 방식을 이용하고 연구해 효율적인 알고리즘을 재탄생시킨 것입니다. Adiantum is a new cipher but it is secure, as long as ChaCha12 and AES-256 are considered secure. Its designers didn't create any new cryptographic primitive, instead they relied on other well-known and thoroughly studied primitives to create a new performant algorithm.

Adiantum은 안드로이드 8(API 레벨 28) 이상부터 사용이 가능합니다. 기본적으로 리눅스 커널 5.0 이상부터 지원하기 때문에 커널 4.19, 4.14와 4.9는 패치가 필요합니다. 안드로이드는 Adiantum을 사용할 수 있는 API를 앱 개발자들에게 제공하고 있지 않습니다; 이 암호 방식은 저가형 단말기에서도 성능에 문제없이 전체 디스크 암호화를 지원할 수 있도록 ROM 개발자나 단말기 벤더사를 고려해 만들어졌기 때문입니다. 이 글이 작성되는 현재까지 안드로이드 애플리케이션에서 이 암호화 방식을 사용할 수 있는 공개된 암호화 라이브러리는 없습니다. AES 명령 세트가있는 장치에서는 AES가 더 빠르게 실행됩니다. 이 경우 Adiantum을 사용하지 않는 것이 좋습니다.  Adiantum is available for Android 9 (API level 28) and higher versions. It is natively supported in Linux kernel 5.0 and onwards, while kernel 4.19, 4.14 & 4.9 need patching.
Android does not provide an API to application developers to use Adiantum; this cipher is to be taken into account and implemented by ROM developers or device vendors, which want to provide full disk encryption without sacrificing performance on low-end devices. At the moment of writing there is no public cryptographic library that implements this cipher to use it on Android applications.
It should be noted that AES runs faster on devices having the AES instruction set. In that case the use of Adiantum is highly discouraged.

### 안드로이드 앱 Apps on Android

#### 운영시스템과 통신 Communication with the Operating System

안드로이드 앱은 Java API를 제공하는 추상 레이어인 안드로이드 프레임워크를 통해 시스템 서비스와 상호작용합니다. 대부분의 이러한 서비스들은 일반적인 자바 메소드 호출에 의해 실행되고 백그라운드에서 실행되는 시스템 서비스에 대한 IPC 호출로 변환됩니다. 시스템 서비스의 예시는 아래와 같습니다. : Android apps interact with system services via the Android Framework, an abstraction layer that offers high-level Java APIs. 대부ㅂThe majority of these services are invoked via normal Java method calls and are translated to IPC calls to system services that are running in the background. Examples of system services include:


- 연결 (Wi-Fi, 블루투스, NFC 등) Connectivity (Wi-Fi, Bluetooth, NFC, etc.)
- 파일 Files
- 카메라 Cameras
- 위치(GPS) Geolocation (GPS)
- 마이크 Microphone

프레임워크는 암호화와 같은 보안 기능도 제공합니다. The framework also offers common security functions, such as cryptography.

API 명세들은 새로운 안드로이드가 발표될 때마다 변경됩니다. 심각한 버그는 수정되고 전 버전의 보안 패치도 이루어집니다. 이 글이 쓰여지는 시점에 가장 오래된 안드로이드 버전은 안드로이드 7.0(API 레벨 24-25) 이고 현재 버전은 안드로이드 9(API 레벨 28) 입니다. The API specifications change with every new Android release. Critical bug fixes and security patches are usually applied to earlier versions as well. The oldest Android version supported at the time of writing is Android 7.0 (API level 24-25) and the current Android version is Android 9 (API level 28).

알아두면 좋은 API 버전들 : Noteworthy API versions:

- 안드로이드 4.2(API 레벨 16) 2012년 11월 (SELinux가 소개됨) Android 4.2 (API level 16) in November 2012 (introduction of SELinux)
- 안드로이드 4.3(API 레벨 18) 2013년 7월 (SELinux가 기본이 됨) Android 4.3 (API level 18) in July 2013 (SELinux became enabled by default)
- 안드로이드 4.4(API 레벨 19) 2013년 10월 (몇 가지 새로운 API 추가, ART 소개됨) Android 4.4 (API level 19) in October 2013 (several new APIs and ART introduced)
- 안드로이드 5.0(API 레벨 21) 2014년 11월 (ART가 기본으로 사용되고 많은 다른 특성이 추가됨) Android 5.0 (API level 21) in November 2014 (ART used by default and many other features added)
- 안드로이드 6.0(API 레벨 23) 2015년 10월 (많은 특성이 추가되고 개선됨, 가령 앱 설치 시 권한을 허용하는 프로세스가 없어지고 앱 실행 시 권한 별로 허용하는 기능 추가)Android 6.0 (API level 23) in October 2015 (many new features and improvements, including granting; detailed permissions setup at run time rather than all or nothing during installation)
- 안드로이드 7.0(API 레벨 24-25) 2016년 8월 (ART의 새로운 JIT 컴파일러 소개) Android 7.0 (API level 24-25) in August 2016 (new JIT compiler on ART)
- 안드로이드 8.0(API 레벨 26-27) 2017년 8월 (많은 보안 개선) Android 8.0 (API level 26-27) in August 2017 (A lot of security improvements)
- 안드로이드 9 (API 레벨 28) 2018년 10월 Android 9 (API level 28) in August 2018.

#### 일반 앱의 리눅스 UID/GID Linux UID/GID for Normal Applications

안드로이드는 앱을 분리시키기 위해 리눅스의 사용자 관리 방식을 채택했습니다. 이런 방식은 전통적인 리눅스 환경에서 동일 사용자가 여러 앱을 실행하는 것과 기존 방식과 다릅니다. 안드로이드는 각각의 안드로이드 앱을 위해 유일한 UID를 만들고 각 프로세스에서 앱을 실행시킵니다. 결과적으로 각각의 앱은 자신의 자원에만 접근할 수 있습니다. 이 보호는 리눅스 커널에서 실행됩니다. Android leverages Linux user management to isolate apps. This approach is different from user management usage in traditional Linux environments, where multiple apps are often run by the same user. Android creates a unique UID for each Android app and runs the app in a separate process. Consequently, each app can access its own resources only. This protection is enforced by the Linux kernel.

일반적으로 앱은 10000에서 99999 범위의 UID가 할당됩니다. 안드로이드 앱은 UID에 기반해 사용자 이름을 할당받습니다. 예를 들어, 앱의 UID가 10188라면 사용자 이름은 `u0_a188`이 됩니다. 만약 앱에서 요청한 권한을 사용자가 허가하면, 관련된 그룹 ID가 앱의 프로세스에 추가됩니다. 예를 들어 아래와 같이 앱의 사용자 ID가 10188이고 그룹 ID는 3003(inet)로 표시된 것처럼요. 이 그룹 ID는 android.permission.INTERNET 권한과 관련있습니다. 아래는 `id` 명령어의 결과입니다. Generally, apps are assigned UIDs in the range of 10000 and 99999. Android apps receive a user name based on their UID. For example, the app with UID 10188 receives the user name `u0_a188`. If the permissions an app requested are granted, the corresponding group ID is added to the app's process. For example, the user ID of the app below is 10188. It belongs to the group ID 3003 (inet). That group is related to android.permission.INTERNET permission. The output of the `id` command is shown below.

```shell
$ id
uid=10188(u0_a188) gid=10188(u0_a188) groups=10188(u0_a188),3003(inet),
9997(everybody),50188(all_a188) context=u:r:untrusted_app:s0:c512,c768
```

그룹 ID와 권한 간 관계는 [frameworks/base/data/etc/platform.xml](http://androidxref.com/7.1.1_r6/xref/frameworks/base/data/etc/platform.xml "platform.xml")에 정의되어 있습니다. The relationship between group IDs and permissions is defined in the file [frameworks/base/data/etc/platform.xml](http://androidxref.com/7.1.1_r6/xref/frameworks/base/data/etc/platform.xml "platform.xml")

```xml
<permission name="android.permission.INTERNET" >
    <group gid="inet" />
</permission>

<permission name="android.permission.READ_LOGS" >
    <group gid="log" />
</permission>

<permission name="android.permission.WRITE_MEDIA_STORAGE" >
    <group gid="media_rw" />
    <group gid="sdcard_rw" />
</permission>
```

#### 앱 샌드박스 The App Sandbox

앱은 안드로이드 애플리케이션 샌드박스 안에서 실행됩니다. 안드로이드 샌드박스는 앱 데이터와 코드 실행을 단말기 내 다른 앱과 분리시킵니다. 이로써 보안을 한 단계 강화할 수 있습니다.  Apps are executed in the Android Application Sandbox, which separates the app data and code execution from other apps on the device. This separation adds a layer of security.

새로운 앱을 설치하면 앱 패키지 이름을 따온 새로운 디렉토리가 `/data/data/[package-name]` 경로에 만들어집니다. 이 디렉토리에 앱 데이터가 저장됩니다. 앱의 UID 권한만 디렉토리를 읽고 쓸 수 있는 리눅스 디렉토리 권한이 설정됩니다. Installation of a new app creates a new directory named after the app package, which results in the following path: `/data/data/[package-name]`. This directory holds the app's data. Linux directory permissions are set such that the directory can be read from and written to only with the app's unique UID.

<img src="Images/Chapters/0x05a/Selection_003.png" alt="Sandbox" width="400">

`/data/data` 폴더에 파일 시스템 권한을 살펴보면 확인할 수 있습니다. 예를 들어 구글 크롬과 캘린더에 각각 디렉토리가 할당되고 다른 사용자 계정으로 권한이 설정되는 것을 알 수 있습니다. We can confirm this by looking at the file system permissions in the `/data/data` folder. For example, we can see that Google Chrome and Calendar are assigned one directory each and run under different user accounts:

```shell
drwx------  4 u0_a97              u0_a97              4096 2017-01-18 14:27 com.android.calendar
drwx------  6 u0_a120             u0_a120             4096 2017-01-19 12:54 com.android.chrome
```

앱 간 공통된 샌드박스를 공유하는 앱을 만들고 싶은 경우 샌드박스를 피할 수 있습니다. 두 앱이 동일한 인증서로 사이닝되고 명확하게 동일한 사용자 ID를 공유(_AndroidManifest.xml_ 파일에서 _sharedUserId_ 를 선언)하면 다른 앱의 데이터 디렉토리에 접근이 가능합니다. 아래는 NFC 앱에서 이 방식을 사용하는 예시입니다. Developers who want their apps to share a common sandbox can sidestep sandboxing . When two apps are signed with the same certificate and explicitly share the same user ID (having the _sharedUserId_ in their _AndroidManifest.xml_ files), each can access the other's data directory. See the following example to achieve this in the NFC app:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
  package="com.android.nfc"
  android:sharedUserId="android.uid.nfc">
```

##### Zygote

`Zygote` 프로세스는 [안드로이드 초기화](https://github.com/dogriffiths/HeadFirstAndroid/wiki/How-Android-Apps-are-Built-and-Run "How Android Apps are run") 중에 시작됩니다. Zygote는 앱을 동작시키기 위한 시스템 서비스입니다. Zygote는 앱 실행에 필요한 주요 라이브러리를 모두 포한한 기본 프로세스입니다. 앱이 실행되는 동안 Zygote는 `/dev/socket/zygote` 소켓을 열고 로컬 클라이언트와 연결을 기다립니다. 연결이 이루어지면 새로울 프로세스를 fork(복사)하고 앱별 코드를 실행하고 로드합니다.  The process `Zygote` starts up during [Android initialization](https://github.com/dogriffiths/HeadFirstAndroid/wiki/How-Android-Apps-are-Built-and-Run "How Android Apps are run"). Zygote is a system service for launching apps. The Zygote process is a "base" process that contains all the core libraries the app needs. Upon launch, Zygote opens the socket `/dev/socket/zygote` and listens for connections from local clients. When it receives a connection, it forks a new process, which then loads and executes the app-specific code.

##### 앱 생명주기 App Lifeycle

안드로이드에서 앱 프로세스의 수명은 운영 시스템이 결정할 수 있습니다. 앱의 컴포넌트가 처음 시작되고 아직 그 앱의 컴포넌트가 실행된 적이 없다면 새로운 리눅스 프로세스가 생성됩니다. 안드로이드는  더 이상 필요하지 않거나 더 중요한 앱을 동작시키기 위해서 메모리 회수가 필요할 때 이 프로세스를 종료할 수도 있습니다. 프로세스 종료 걸정은 프로세스와 사용자 간 상호작용 상태와 관련있습니다. 일반적으로 프로세스는 4가지 상태중 하나일 수 있습니다.  In Android, the lifetime of an app process is controlled by the operating system. A new Linux process is created when an app component is started and the same app doesn’t yet have any other components running. Android may kill this process when the latter is no longer necessary or when reclaiming memory is necessary to run more important apps. The decision to kill a process is primarily related to the state of the user's interaction with the process. In general, processes can be in one of four states.

- A foreground process (e.g., an activity running at the top of the screen or a running BroadcastReceive)
- A visible process is a process that the user is aware of, so killing it would have a noticeable negative impact on user experience. One example is running an activity that's visible to the user on-screen but not in the foreground.

- A service process is a process hosting a service that has been started with the `startService` method. Though these processes aren't directly visible to the user, they are generally things that the user cares about (such as background network data upload or download), so the system will always keep such processes running unless there's insufficient memory to retain all foreground and visible processes.
- A cached process is a process that's not currently needed, so the system is free to kill it when memory is needed.
Apps must implement callback methods that react to a number of events; for example, the `onCreate` handler is called when the app process is first created. Other callback methods include `onLowMemory`, `onTrimMemory` and `onConfigurationChanged`.

##### App Bundles

Android applications can be shipped in two forms: the Android Package Kit (APK) file or an [Android App Bundle](https://developer.android.com/guide/app-bundle "Android App Bundle") (.aab). Android App Bundles provide all the resources necessary for an app, but defer the generation of the APK and its signing to Google Play. App Bundles are signed binaries which contain the code of the app in several modules. The base module contains the core of the application. The base module can be extended with various modules which contain new enrichments/functionalities for the app as further explained on the [developer documentation for app bundle](https://developer.android.com/guide/app-bundle "Documentation on App Bundle").
If you have an Android App Bundle, you can best use the [bundletool](https://developer.android.com/studio/command-line/bundletool "bundletool") command line tool from Google to build unsigned APKs in order to use the existing tooling on the APK. You can create an APK from an AAB file by running the following command:

```shell

$ bundletool build-apks --bundle=/MyApp/my_app.aab --output=/MyApp/my_app.apks

```

If you want to create signed APKs ready for deployment to a test device, use:

```shell
$ bundletool build-apks --bundle=/MyApp/my_app.aab --output=/MyApp/my_app.apks
--ks=/MyApp/keystore.jks
--ks-pass=file:/MyApp/keystore.pwd
--ks-key-alias=MyKeyAlias
--key-pass=file:/MyApp/key.pwd
```

We recommend that you test both the APK with and without the additional modules, so that it becomes clear whether the additional modules introduce and/or fix security issues for the base module.

##### Android Manifest

Every app has an Android Manifest file, which embeds content in binary XML format. The standard name of this file is AndroidManifest.xml. It is located in the root directory of the app’s Android Package Kit (APK) file.

The manifest file describes the app structure, its components (activities, services, content providers, and intent receivers), and requested permissions. It also contains general app metadata, such as the app's icon, version number, and theme. The file may list other information, such as compatible APIs (minimal, targeted, and maximal SDK version) and the [kind of storage it can be installed on (external or internal)](https://developer.android.com/guide/topics/data/install-location.html "Define app install location").

Here is an example of a manifest file, including the package name (the convention is a reversed URL, but any string is acceptable). It also lists the app version, relevant SDKs, required permissions, exposed content providers, broadcast receivers used with intent filters and a description of the app and its activities:

```xml
<manifest
    package="com.owasp.myapplication"
    android:versionCode="0.1" >

    <uses-sdk android:minSdkVersion="12"
        android:targetSdkVersion="22"
        android:maxSdkVersion="25" />

    <uses-permission android:name="android.permission.INTERNET" />

    <provider
        android:name="com.owasp.myapplication.myProvider"
        android:exported="false" />

    <receiver android:name=".myReceiver" >
        <intent-filter>
            <action android:name="com.owasp.myapplication.myaction" />
        </intent-filter>
    </receiver>

    <application
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/Theme.Material.Light" >
        <activity
            android:name="com.owasp.myapplication.MainActivity" >
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
            </intent-filter>
        </activity>
    </application>
</manifest>
```

The full list of available manifest options is in the official [Android Manifest file documentation](https://developer.android.com/guide/topics/manifest/manifest-intro.html "Android Developer Guide for Manifest").

#### App Components

Android apps are made of several high-level components. The main components are:

- Activities
- Fragments
- Intents
- Broadcast receivers
- Content providers and services

All these elements are provided by the Android operating system, in the form of predefined classes available through APIs.

##### Activities

Activities make up the visible part of any app. There is one activity per screen, so an app with three different screens implements three different activities. Activities are declared by extending the Activity class. They contain all user interface elements: fragments, views, and layouts.

Each activity needs to be declared in the Android Manifest with the following syntax:

```xml
<activity android:name="ActivityName">
</activity>
```

Activities not declared in the manifest can't be displayed, and attempting to launch them will raise an exception.

Like apps, activities have their own life cycle and need to monitor system changes to handle them. Activities can be in the following states: active, paused, stopped, and inactive. These states are managed by the Android operating system. Accordingly, activities can implement the following event managers:

- onCreate
- onSaveInstanceState
- onStart
- onResume
- onRestoreInstanceState
- onPause
- onStop
- onRestart
- onDestroy

An app may not explicitly implement all event managers, in which case default actions are taken. Typically, at least the `onCreate` manager is overridden by the app developers. This is how most user interface components are declared and initialized. `onDestroy` may be overridden when resources (like network connections or connections to databases) must be explicitly released or specific actions must occur when the app shuts down.

##### Fragments

A fragment represents a behavior or a portion of the user interface within the activity. Fragments were introduced Android with the version Honeycomb 3.0 (API level 11).

Fragments are meant to encapsulate parts of the interface to facilitate re-usability and adaptation to different screen sizes. Fragments are autonomous entities in that they include all their required components (they have their own layout, buttons, etc.). However, they must be integrated with activities to be useful: fragments can't exist on their own. They have their own life cycle, which is tied to the life cycle of the Activities that implement them.

Because fragments have their own life cycle, the Fragment class contains event managers that can be redefined and extended. These event managers included onAttach, onCreate, onStart, onDestroy and onDetach. Several others exist; the reader should refer to the [Android Fragment specification](https://developer.android.com/reference/android/app/Fragment.html "Fragment Class") for more details.

Fragments can be easily implemented by extending the Fragment class provided by Android:

```Java
public class myFragment extends Fragment {
    ...
}
```

Fragments don't need to be declared in manifest files because they depend on activities.

To manage its fragments, an activity can use a Fragment Manager (FragmentManager class). This class makes it easy to find, add, remove, and replace associated fragments.

Fragment Managers can be created via the following:

```Java
FragmentManager fm = getFragmentManager();
```

Fragments don't necessarily have a user interface; they can be a convenient and efficient way to manage background operations pertaining to the app's user interface. A fragment may be declared persistent so that if the system preserves its state even if its Activity is destroyed.

##### Inter-Process Communication

As we've already learned, every Android process has its own sandboxed address space. Inter-process communication facilities allow apps to exchange signals and data securely. Instead of relying on the default Linux IPC facilities, Android's IPC is based on Binder, a custom implementation of OpenBinder. Most Android system services and all high-level IPC services depend on Binder.

The term *Binder* stands for a lot of different things, including:

- Binder Driver: the kernel-level driver
- Binder Protocol: low-level ioctl-based protocol used to communicate with the binder driver
- IBinder Interface: a well-defined behavior that Binder objects implement
- Binder object: generic implementation of the IBinder interface
- Binder service: implementation of the Binder object; for example, location service, and sensor service
- Binder client: an object using the Binder service

The Binder framework includes a client-server communication model. To use IPC, apps call IPC methods in proxy objects. The proxy objects transparently *marshall* the call parameters into a *parcel* and send a transaction to the Binder server, which is implemented as a character driver (/dev/binder). The server holds a thread pool for handling incoming requests and delivers messages to the destination object. From the perspective of the client app, all of this seems like a regular method call—all the heavy lifting is done by the Binder framework.

<img src="Images/Chapters/0x05a/binder.jpg" alt="Binder Overview" width="400">

*Binder Overview - Image source: [Android Binder by Thorsten Schreiber](https://www.nds.rub.de/media/attachments/files/2011/10/main.pdf "Android Binder")*

Services that allow other applications to bind to them are called *bound services*. These services must provide an IBinder interface to clients. Developers use the Android Interface Descriptor Language (AIDL) to write interfaces for remote services.

Servicemanager is a system daemon that manages the registration and lookup of system services. It maintains a list of name/Binder pairs for all registered services. Services are added with `addService` and retrieved by name with the static `getService` method in `android.os.ServiceManager`:

```java
  public static IBinder getService(String name)
```

You can query the list of system services with the `service list` command.

```shell
$ adb shell service list
Found 99 services:
0 carrier_config: [com.android.internal.telephony.ICarrierConfigLoader]
1 phone: [com.android.internal.telephony.ITelephony]
2 isms: [com.android.internal.telephony.ISms]
3 iphonesubinfo: [com.android.internal.telephony.IPhoneSubInfo]
```

#### Intents

*Intent messaging* is an asynchronous communication framework built on top of Binder. This framework allows both point-to-point and publish-subscribe messaging. An *Intent* is a messaging object that can be used to request an action from another app component. Although intents facilitate inter-component communication in several ways, there are three fundamental use cases:

- Starting an activity
  - An activity represents a single screen in an app. You can start a new instance of an activity by passing an intent to `startActivity`. The intent describes the activity and carries necessary data.
- Starting a service
  - A Service is a component that performs operations in the background, without a user interface. With Android 5.0 (API level 21) and later, you can start a service with JobScheduler.
- Delivering a broadcast
  - A broadcast is a message that any app can receive. The system delivers broadcasts for system events, including system boot and charging initialization. You can deliver a broadcast to other apps by passing an intent to `sendBroadcast` or `sendOrderedBroadcast`.

There are two types of intents. Explicit intents name the component that will be started (the fully qualified class name). For instance:

```Java
    Intent intent = new Intent(this, myActivity.myClass);
```

Implicit intents are sent to the OS to perform a given action on a given set of data (The URL of the OWASP website in our example below). It is up to the system to decide which app or class will perform the corresponding service. For instance:

```Java
    Intent intent = new Intent(Intent.MY_ACTION, Uri.parse("https://www.owasp.org"));
```

An *intent filter* is an expression in Android Manifest files that specifies the type of intents the component would like to receive. For instance, by declaring an intent filter for an activity, you make it possible for other apps to directly start your activity with a certain kind of intent. Likewise, your activity can only be started with an explicit intent if you don't declare any intent filters for it.

Android uses intents to broadcast messages to apps (such as an incoming call or SMS) important power supply information (low battery, for example), and network changes (loss of connection, for instance). Extra data may be added to intents (through `putExtra`/`getExtras`).

Here is a short list of intents sent by the operating system. All constants are defined in the Intent class, and the whole list is in the official Android documentation:

- ACTION_CAMERA_BUTTON
- ACTION_MEDIA_EJECT
- ACTION_NEW_OUTGOING_CALL
- ACTION_TIMEZONE_CHANGED

To improve security and privacy, a Local Broadcast Manager is used to send and receive intents within an app without having them sent to the rest of the operating system. This is very useful for ensuring that sensitive and private data don't leave the app perimeter (geolocation data for instance).

##### Broadcast Receivers

Broadcast Receivers are components that allow apps to receive notifications from other apps and from the system itself. With them, apps can react to events (internal, initiated by other apps, or initiated by the operating system). They are generally used to update user interfaces, start services, update content, and create user notifications.

There are two ways to make a Broadcast Receiver known to the system. One way is to declare it in the Android Manifest file. The manifest should specify an association between the Broadcast Receiver and an intent filter to indicate the actions the receiver is meant to listen for.

An example Broadcast Receiver declaration with an intent filter in a manifest:

```xml
<receiver android:name=".myReceiver" >
    <intent-filter>
        <action android:name="com.owasp.myapplication.MY_ACTION" />
    </intent-filter>
</receiver>
```

Please note that in this example, the Broadcast Receiver does not include the [`android:exported`](https://developer.android.com/guide/topics/manifest/receiver-element "receiver element") attribute. As at least one filter was defined, the default value will be set to "true". In absence of any filters, it will be set to "false".

The other way is to create the receiver dynamically in code and register it with the [`Context.registerReceiver`](https://developer.android.com/reference/android/content/Context.html#registerReceiver(android.content.BroadcastReceiver,%2520android.content.IntentFilter) "Context.registerReceiver") method.

An example of registering a Broadcast Receiver dynamically:

```Java
// Define a broadcast receiver
myReceiver = new BroadcastReceiver() {
    @Override
    public void onReceive(Context context, Intent intent) {
        Log.d(TAG, "Intent received by myReceiver");
    }
};
// Define an intent filter with actions that the broadcast receiver listens for
IntentFilter intentFilter = new IntentFilter();
intentFilter.addAction("com.owasp.myapplication.MY_ACTION");
// To register the broadcast receiver
registerReceiver(myReceiver, intentFilter);
// To un-register the broadcast receiver
unregisterReceiver(myReceiver);
```

Note that the system starts an app with the registered receiver automatically when a relevant intent is raised.

According to [Broadcasts Overview](https://developer.android.com/guide/components/broadcasts "Broadcasts Overview"), a broadcast is considered "implicit" if it does not target an app specifically. After receiving an implicit broadcast, Android will list all apps that have registered a given action in their filters. If more than one app has registered for the same action, Android will prompt the user to select from the list of available apps.

An interesting feature of Broadcast Receivers is that they can be prioritized; this way, an intent will be delivered to all authorized receivers according to their priority. A priority can be assigned to an intent filter in the manifest via the `android:priority` attribute as well as programmatically via the [`IntentFilter.setPriority`](https://developer.android.com/reference/android/content/IntentFilter#setPriority(int) "IntentFilter.setPriority") method. However, note that receivers with the same priority will be [run in an arbitrary order](https://developer.android.com/guide/components/broadcasts.html#sending-broadcasts "Sending Broadcasts").

If your app is not supposed to send broadcasts across apps, use a Local Broadcast Manager ([`LocalBroadcastManager`](https://developer.android.com/reference/androidx/localbroadcastmanager/content/LocalBroadcastManager.html "LocalBroadcastManager")). They can be used to make sure intents are received from the internal app only, and any intent from any other app will be discarded. This is very useful for improving security and the efficiency of the app, as no interprocess communication is involved. However, please note that the `LocalBroadcastManager` class is [deprecated](https://developer.android.com/reference/androidx/localbroadcastmanager/content/LocalBroadcastManager.html "LocalBroadcastManager") and Google recommends using alternatives such as [`LiveData`](https://developer.android.com/reference/androidx/lifecycle/LiveData.html "LiveData").

For more security considerations regarding Broadcast Receiver, see [Security Considerations and Best Practices](https://developer.android.com/guide/components/broadcasts.html#security-and-best-practices "Security Considerations and Best Practices").

###### Implicit Broadcast Receiver Limitiation

According to [Background Optimizations](https://developer.android.com/topic/performance/background-optimization "Background Optimizations"), apps targeting Android 7.0 (API level 24) or higher no longer receive `CONNECTIVITY_ACTION` broadcast unless they register their Broadcast Receivers with `Context.registerReceiver()`. The system does not send `ACTION_NEW_PICTURE` and `ACTION_NEW_VIDEO` broadcasts as well.

According to [Background Execution Limits](https://developer.android.com/about/versions/oreo/background.html#broadcasts "Background Execution Limits"), apps that target Android 8.0 (API level 26) or higher can no longer register Broadcast Receivers for implicit broadcasts in their manifest, except for those listed in [Implicit Broadcast Exceptions](https://developer.android.com/guide/components/broadcast-exceptions "Implicit Broadcast Exceptions"). The Broadcast Receivers created at runtime by calling `Context.registerReceiver` are not affected by this limitation.

According to [Changes to System Broadcasts](https://developer.android.com/guide/components/broadcasts#changes-system-broadcasts "Changes to System Broadcasts"), beginning with Android 9 (API level 28), the `NETWORK_STATE_CHANGED_ACTION` broadcast doesn't receive information about the user's location or personally identifiable data.

##### Content Providers

Android uses SQLite to store data permanently: as with Linux, data is stored in files. SQLite is a light, efficient, open source relational data storage technology that does not require much processing power, which makes it ideal for mobile use. An entire API with specific classes (Cursor, ContentValues, SQLiteOpenHelper, ContentProvider, ContentResolver, etc.) is available.
SQLite is not run as a separate process; it is part of the app.
By default, a database belonging to a given app is accessible to this app only. However, content providers offer a great mechanism for abstracting data sources (including databases and flat files); they also provide a standard and efficient mechanism to share data between apps, including native apps. To be accessible to other apps, a content provider needs to be explicitly declared in the manifest file of the app that will share it. As long as content providers aren't declared, they won't be exported and can only be called by the app that creates them.

content providers are implemented through a URI addressing scheme: they all use the content:// model. Regardless of the type of sources (SQLite database, flat file, etc.), the addressing scheme is always the same, thereby abstracting the sources and offering the developer a unique scheme. Content Providers offer all regular database operations: create, read, update, delete. That means that any app with proper rights in its manifest file can manipulate the data from other apps.

##### Services

Services are Android OS components (based on the Service class) that perform tasks in the background (data processing, starting intents, and notifications, etc.) without presenting a user interface. Services are meant to run processes long-term. Their system priorities are lower than those of active apps and higher than those of inactive apps. Therefore, they are less likely to be killed when the system needs resources, and they can be configured to automatically restart when enough resources become available. Activities are executed in the main app thread. They are great candidates for running asynchronous tasks.

##### Permissions

Because Android apps are installed in a sandbox and initially can't access user information and system components (such as the camera and the microphone), Android provides a system with a predefined set of permissions for certain tasks that the app can request.
For example, if you want your app to use a phone's camera, you have to request the `android.permission.CAMERA` permission.
Prior to Android 6.0 (API level 23), all permissions an app requested were granted at installation. From API level 23 onwards, the user must approve some permissions requests during app execution.

###### Protection Levels

Android permissions are ranked on the basis of the protection level they offer and divided into four different categories:

- *Normal*: the lower level of protection. It gives the apps access to isolated application-level features with minimal risk to other apps, the user, or the system. It is granted during app installation and is the default protection level:
Example: `android.permission.INTERNET`
- *Dangerous*: This permission allows the app to perform actions that might affect the user’s privacy or the normal operation of the user’s device. This level of permission may not be granted during installation; the user must decide whether the app should have this permission.
Example: `android.permission.RECORD_AUDIO`
- *Signature*: This permission is granted only if the requesting app has been signed with the same certificate as the app that declared the permission. If the signature matches, the permission is automatically granted.
Example: `android.permission.ACCESS_MOCK_LOCATION`
- *SystemOrSignature*: This permission is granted only to apps embedded in the system image or signed with the same certificate that the app that declared the permission was signed with.
Example: `android.permission.ACCESS_DOWNLOAD_MANAGER`

###### Requesting Permissions

Apps can request permissions for the protection levels Normal, Dangerous, and Signature by including `<uses-permission />` tags into their manifest.
The example below shows an AndroidManifest.xml sample requesting permission to read SMS messages:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.permissions.sample" ...>

    <uses-permission android:name="android.permission.RECEIVE_SMS" />
    <application>...</application>
</manifest>
```

###### Declaring Permissions

Apps can expose features and content to other apps installed on the system. To restrict access to its own components, it can either use any of Android’s [predefined permissions](https://developer.android.com/reference/android/Manifest.permission.html "predefined permissions") or define its own. A new permission is declared with the `<permission>` element.
The example below shows an app declaring a permission:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.permissions.sample" ...>

    <permission
    android:name="com.permissions.sample.ACCESS_USER_INFO"
    android:protectionLevel="signature" />
    <application>...</application>
</manifest>
```

The above code defines a new permission named `com.permissions.sample.ACCESS_USER_INFO` with the protection level `Signature`. Any components protected with this permission would be accessible only by apps signed with the same developer certificate.

###### Enforcing Permissions on Android Components

Android components can be protected with permissions. Activities, Services, Content Providers, and Broadcast Receivers—all can use the permission mechanism to protect their interfaces.
Permissions can be enforced on *Activities*, *Services*, and *Broadcast Receivers* by adding the attribute *android:permission* to the respective component tag in AndroidManifest.xml:

```xml
<receiver
    android:name="com.permissions.sample.AnalyticsReceiver"
    android:enabled="true"
    android:permission="com.permissions.sample.ACCESS_USER_INFO">
    ...
</receiver>
```

*Content Providers* are a little different. They support a separate set of permissions for reading, writing, and accessing the content provider with a content URI.

- `android:writePermission`, `android:readPermission`: the developer can set separate permissions for reading or writing.
- `android:permission`: general permission that will control reading and writing to the content provider.
- `android:grantUriPermissions`: `"true"` if the content provider can be accessed with a content URI (the access temporarily bypasses the restrictions of other permissions), and `"false"` otherwise.

### Signing and Publishing Process

Once an app has been successfully developed, the next step is to publish and share it with others. However, apps can't simply be added to a store and shared, for several reasons—they must be signed. The cryptographic signature serves as a verifiable mark placed by the developer of the app. It identifies the app’s author and ensures that the app has not been modified since its initial distribution.

#### Signing Process

During development, apps are signed with an automatically generated certificate. This certificate is inherently insecure and is for debugging only. Most stores don't accept this kind of certificate for publishing; therefore, a certificate with more secure features must be created.
When an application is installed on the Android device, the Package Manager ensures that it has been signed with the certificate included in the corresponding APK. If the certificate's public key matches the key used to sign any other APK on the device, the new APK may share a UID with the pre-existing APK. This facilitates interactions between applications from a single vendor. Alternatively, specifying security permissions for the Signature protection level is possible; this will restrict access to applications that have been signed with the same key.

#### APK Signing Schemes

Android supports three application signing schemes. Starting with Android 9 (API level 28), APKs can be verified with APK Signature Scheme v3 (v3 scheme), APK Signature Scheme v2 (v2 scheme) or JAR signing (v1 scheme). For Android 7.0 (API level 24) and above, APKs can be verified with the APK Signature Scheme v2 (v2 scheme) or JAR signing (v1 scheme). For backwards compatibility, an APK can be signed with multiple signature schemes in order to make the app run on both newer and older SDK versions. [Older platforms ignore v2 signatures and verify v1 signatures only](https://source.android.com/security/apksigning/ "APK Signing ").

##### JAR Signing (v1 Scheme)

The original version of app signing implements the signed APK as a standard signed JAR, which must contain all the entries in `META-INF/MANIFEST.MF`. All files must be signed with a common certificate. This scheme does not protect some parts of the APK, such as ZIP metadata. The drawback of this scheme is that the APK verifier needs to process untrusted data structures before applying the signature, and the verifier discards data the data structures don't cover. Also, the APK verifier must decompress all compressed files, which takes considerable time and memory.

##### APK Signature Scheme (v2 Scheme)

With the APK signature scheme, the complete APK is hashed and signed, and an APK Signing Block is created and inserted into the APK. During validation, the v2 scheme checks the signatures of the entire APK file. This form of APK verification is faster and offers more comprehensive protection against modification. You can see the [APK signature verification process for v2 Scheme](https://source.android.com/security/apksigning/v2#verification "APK Signature verification process") below.

<img src="Images/Chapters/0x05a/apk-validation-process.png" alt="Android Software Stack" width="450">

#### APK Signature Scheme (v3 Scheme)

The v3 APK Signing Block format is the same as v2. V3 adds information about the supported SDK versions and a proof-of-rotation struct to the APK signing block. In Android 9 (API level 28) and higher, APKs can be verified according to APK Signature Scheme v3, v2 or v1 scheme. Older platforms ignore v3 signatures and try to verify v2 then v1 signature.

The proof-of-rotation attribute in the signed-data of the signing block consists of a singly-linked list, with each node containing a signing certificate used to sign previous versions of the app. To make backward compatibility work, the old signing certificates sign the new set of certificates, thus providing each new key with evidence that it should be as trusted as the older key(s).
It is no longer possible to sign APKs independently, because the proof-of-rotation structure must have the old signing certificates signing the new set of certificates, rather than signing them one-by-one. You can see the [APK signature v3 scheme verification process](https://source.android.com/security/apksigning/v3 "APK Signature v3 scheme verification process") below.

<img src="Images/Chapters/0x05a/apk-validation-process-v3-scheme.png" alt="apk-validation-process-v3-scheme" width="450">

##### Creating Your Certificate

Android uses public/private certificates to sign Android apps (.apk files). Certificates are bundles of information; in terms of security, keys are the most important type of this information Public certificates contain users' public keys, and private certificates contain users' private keys. Public and private certificates are linked. Certificates are unique and can't be re-generated. Note that if a certificate is lost, it cannot be recovered, so updating any apps signed with that certificate becomes impossible.
App creators can either reuse an existing private/public key pair that is in an available KeyStore or generate a new pair.
In the Android SDK, a new key pair is generated with the `keytool` command. The following command creates a RSA key pair with a key length of 2048 bits and an expiry time of 7300 days = 20 years. The generated key pair is stored in the file 'myKeyStore.jks', which is in the current directory):

```shell
$ keytool -genkey -alias myDomain -keyalg RSA -keysize 2048 -validity 7300 -keystore myKeyStore.jks -storepass myStrongPassword
```

Safely storing your secret key and making sure it remains secret during its entire life cycle is of paramount importance. Anyone who gains access to the key will be able to publish updates to your apps with content that you don't control (thereby adding insecure features or accessing shared content with signature-based permissions). The trust that a user places in an app and its developers is based totally on such certificates; certificate protection and secure management are therefore vital for reputation and customer retention, and secret keys must never be shared with other individuals. Keys are stored in a binary file that can be protected with a password; such files are referred to as 'KeyStores'. KeyStore passwords should be strong and known only to the key creator. For this reason, keys are usually stored on a dedicated build machine that developers have limited access to.
An Android certificate must have a validity period that's longer than that of the associated app (including updated versions of the app). For example, Google Play will require certificates to remain valid until Oct 22nd, 2033 at least.

##### Signing an Application

The goal of the signing process is to associate the app file (.apk) with the developer's public key. To achieve this, the developer calculates a hash of the APK file and encrypts it with their own private key. Third parties can then verify the app's authenticity (e.g., the fact that the app really comes from the user who claims to be the originator) by decrypting the encrypted hash with the author’s public key and verifying that it matches the actual hash of the APK file.

Many Integrated Development Environments (IDE) integrate the app signing process to make it easier for the user. Be aware that some IDEs store private keys in clear text in configuration files; double-check this in case others are able to access such files and remove the information if necessary.
Apps can be signed from the command line with the 'apksigner' tool provided by the Android SDK (API level 24 and higher). It is located at `[SDK-Path]/build-tools/[version]`. For API 24.0.2 and below, you can use 'jarsigner', which is part of the Java JDK. Details about the whole process can be found in official Android documentation; however, an example is given below to illustrate the point.

```shell
$ apksigner sign --out mySignedApp.apk --ks myKeyStore.jks myUnsignedApp.apk
```

In this example, an unsigned app ('myUnsignedApp.apk') will be signed with a private key from the developer KeyStore 'myKeyStore.jks' (located in the current directory). The app will become a signed app called 'mySignedApp.apk' and will be ready to release to stores.

###### Zipalign

The `zipalign` tool should always be used to align the APK file before distribution. This tool aligns all uncompressed data (such as images, raw files, and 4-byte boundaries) within the APK that helps improve memory management during app run time.

> Zipalign must be used before the APK file is signed with apksigner.

#### Publishing Process

Distributing apps from anywhere (your own site, any store, etc.) is possible because the Android ecosystem is open. However, Google Play is the most well-known, trusted, and popular store, and Google itself provides it. Amazon Appstore is the trusted default store for Kindle devices. If users want to install third-party apps from a non-trusted source, they must explicitly allow this with their device security settings.

Apps can be installed on an Android device from a variety of sources: locally via USB, via Google's official app store (Google Play Store) or from alternative stores.

Whereas other vendors may review and approve apps before they are actually published, Google will simply scan for known malware signatures; this minimizes the time between the beginning of the publishing process and public app availability.

Publishing an app is quite straightforward; the main operation is making the signed APK file downloadable. On Google Play, publishing starts with account creation and is followed by app delivery through a dedicated interface. Details are available at [the official Android documentation](https://developer.android.com/distribute/googleplay/start.html "Review the checklists to plan your launch").

### Android Application Attack surface

The Android application attack surface consists of all components of the application, including the supportive material necessary to release the app and to support its functioning. The Android application may be vulnerable to attack if it does not:

- Validate all input by means of IPC communication or URL schemes, see also:
  - [Testing for Sensitive Functionality Exposure Through IPC](0x05h-Testing-Platform-Interaction.md#testing-for-sensitive-functionality-exposure-through-ipc-mstg-platform-4 "Testing for Sensitive Functionality Exposure Through IPC")
  - [Testing Custom URL Schemes](0x05h-Testing-Platform-Interaction.md#testing-custom-url-schemes-mstg-platform-3 "Testing Custom URL Schemes")
- Validate all input by the user in input fields.
- Validate the content loaded inside a WebView, see also:
  - [Testing JavaScript Execution in WebViews](0x05h-Testing-Platform-Interaction.md#testing-javascript-execution-in-webviews-mstg-platform-5 "Testing JavaScript Execution in WebViews")
  - [Testing WebView Protocol Handlers](0x05h-Testing-Platform-Interaction.md#testing-webview-protocol-handlers-mstg-platform-6 "Testing WebView Protocol Handlers")
  - [Determining Whether Java Objects Are Exposed Through WebViews](0x05h-Testing-Platform-Interaction.md#determining-whether-java-objects-are-exposed-through-webviews-mstg-platform-7 "Determining Whether Java Objects Are Exposed Through WebViews")
- Securely communicate with backend servers or is susceptible to man-in-the-middle attacks between the server and the mobile application, see also:
  - [Testing Network Communication](0x04f-Testing-Network-Communication.md#testing-network-communication "Testing Network Communication")
  - [Android Network APIs](0x05g-Testing-Network-Communication.md#android-network-apis "Android Network APIs")
- Securely stores all local data, or loads untrusted data from storage, see also:
  - [Data Storage on Android](0x05d-Testing-Data-Storage.md#data-storage-on-android "Data Storage on Android")
- Protect itself against compromised environments, repackaging or other local attacks, see also:
  - [Android Anti-Reversing Defenses](0x05j-Testing-Resiliency-Against-Reverse-Engineering.md#android-anti-reversing-defenses "Android Anti-Reversing Defenses")
