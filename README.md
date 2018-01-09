# vqfx10k-light-4qfx-2srv
* このリポジトリは [vqfx10k-vagrant](https://github.com/Juniper/vqfx10k-vagrant) ベースにしています
* `vagrant up` を実行すると、4つの vQFX10k インスタンスと2つのサーバインスタンスが起動します
* 4つの vQFX10k インスタンスの論理配線は [Example: Configuring BGP Local Preference](https://www.juniper.net/documentation/en_US/junos/topics/topic-map/bgp-local-preference.html) に記載してあるトポロジに従っています
  * また、2つのサーバインスタンスは R1 と R4 に接続されています
