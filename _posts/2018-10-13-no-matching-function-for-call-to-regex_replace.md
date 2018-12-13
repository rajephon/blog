---
title: "error: no matching function for call to ‘regex_replace(std::string&, std::regex&, const char [1])’"
date: 2018-10-13 05:32:54 +0900
categories: Amazon AWS
layout: post
comments: true
pinned: true
---

## 문제
아마존 인스턴스에서 make 명령어로 빌드 시 에러 발생

```bash
[ 16%] Building CXX object CMakeFiles/YKSConverter.dir/YKSConverter/MF2TT2MF/MF2TT2MF.cpp.o
/YKSConverter/YKSConverter/MF2TT2MF/MF2TT2MF.cpp: In member function ‘std::vector<std::shared_ptr<YKS::TE::TrackEvent> > YKS::MF2TT2MF::_parseTrack(std::string, int)’:
/YKSConverter/YKSConverter/MF2TT2MF/MF2TT2MF.cpp:117:52: error: no matching function for call to ‘regex_replace(std::string&, std::regex&, const char [1])’
     track = std::regex_replace(track, rgxFilter, "");
                                                    ^
/YKSConverter/YKSConverter/MF2TT2MF/MF2TT2MF.cpp:117:52: note: candidates are:
In file included from /usr/include/c++/4.8.5/regex:62:0,
                 from /YKSConverter/YKSConverter/MF2TT2MF/MF2TT2MF.cpp:8:
/usr/include/c++/4.8.5/bits/regex.h:2162:5: note: template<class _Out_iter, class _Bi_iter, class _Rx_traits, class _Ch_type> _Out_iter std::regex_replace(_Out_iter, _Bi_iter, _Bi_iter, const std::basic_regex<_Ch_type, _Rx_traits>&, const std::basic_string<_Ch_type>&, std::regex_constants::match_flag_type)
     regex_replace(_Out_iter __out, _Bi_iter __first, _Bi_iter __last,
     ^
/usr/include/c++/4.8.5/bits/regex.h:2162:5: note:   template argument deduction/substitution failed:
/YKSConverter/YKSConverter/MF2TT2MF/MF2TT2MF.cpp:117:52: note:   deduced conflicting types for parameter ‘_Bi_iter’ (‘std::basic_regex<char>’ and ‘const char*’)
     track = std::regex_replace(track, rgxFilter, "");
                                                    ^
In file included from /usr/include/c++/4.8.5/regex:62:0,
                 from /YKSConverter/YKSConverter/MF2TT2MF/MF2TT2MF.cpp:8:
/usr/include/c++/4.8.5/bits/regex.h:2182:5: note: template<class _Rx_traits, class _Ch_type> std::basic_string<_Ch_type> std::regex_replace(const std::basic_string<_Ch_type>&, const std::basic_regex<_Ch_type, _Rx_traits>&, const std::basic_string<_Ch_type>&, std::regex_constants::match_flag_type)
     regex_replace(const basic_string<_Ch_type>& __s,
     ^
/usr/include/c++/4.8.5/bits/regex.h:2182:5: note:   template argument deduction/substitution failed:
/YKSConverter/YKSConverter/MF2TT2MF/MF2TT2MF.cpp:117:52: note:   mismatched types ‘const std::basic_string<_Ch_type>’ and ‘const char [1]’
     track = std::regex_replace(track, rgxFilter, "");
                                                    ^
make[2]: *** [CMakeFiles/YKSConverter.dir/YKSConverter/MF2TT2MF/MF2TT2MF.cpp.o] Error 1
make[1]: *** [CMakeFiles/YKSConverter.dir/all] Error 2
make: *** [all] Error 2
```

## 원인
gcc 버전이 낮아 C++11 regex를 사용할 수 없다.
gcc 4.9 이상 버전으로 빌드해야 한다.

## 해결방안
기존 버전의 gcc를 지우고, 새 버전을 설치한다. `gcc`라는 이름으로 있는 패키지는 4.8.5버전이며, `gcc72`로 설치하면 7.2.1버전을 설치할 수 있다.

```bash
sudo yum install gcc72 gcc72-c++
```

혹은 직접 소스를 내려받아 빌드하여 설치가 가능하다. 다만 시간이 매우 오래 걸리므로, 고성능 인스턴스 사용자에게만 추천한다.

```bash
suto yum remove gcc
sudo yum install libmpc-devel mpfr-devel gmp-devel

cd ~/
mkdir Downloads
cd Downloads
wget http://ftp.mirrorservice.org/sites/sourceware.org/pub/gcc/releases/gcc-8.2.0/gcc-8.2.0.tar.gz
tar -zxvf gcc-8.2.0.tar.bz2

cd gcc-8.2.0
./contrib/download_prerequisites # bzip2가 없을 경우 sudo yum install bzip2
./configure --disable-multilib --enable-languages=c,c++
make -j 4 # thread 갯수
sudo make install
```
