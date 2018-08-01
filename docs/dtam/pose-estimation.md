## カメラの位置姿勢推定

### 基本的な考え方
最終的には世界座標系から見た現在のカメラの位置姿勢\\(T_{wl}\\)を求めたい．  

そこで  

1. 現在のフレームと一つ前のフレームの画像全体の輝度値の差の最小化により，カメラの姿勢変化（3DOF）のみ求める．  
2. 1で得られたカメラ位置姿勢を初期値とし，Denseなモデルから得られた仮想画像と現在のフレームの輝度値の差を最小化することにより，現在のカメラの位置姿勢（6DOF）を求める．  

現在のフレームと一つ前のフレームはMotion blurが似ていると仮定することで，1によりblurに対して頑健な初期推定が可能である．  
1も2もImage pyramidを用いたLucas-Kanadeベースの非線形最適化でカメラの位置姿勢を推定する．

### 1. カメラの姿勢のみの変化推定
リアルタイムのパノラマ合成であり，[1]の手法を用いている．  

2フレームにおけるカメラの光学中心が変化しないことを想定し，カメラの姿勢変化 \\(\mathbb{SO}(3)\\)を推定する．  
Direct methodの一般的な非線形最適化を用いており，式は論文を参照すること．  

Denseな手法なので，画像全体の輝度値の差を最小化する．  
この最小化をリアルタイムに行うためにシェーディング言語を活用している．  

非線形最適化の際に毎ステップの計算が必要なJacobianはGPUで計算する．
これを画像としてCPUに返し，CPU側でコレスキー分解を用いて解を求める．  

### 2. カメラの姿勢変化
Dense modelから得られた仮想画像と現在のフレームの輝度値の差の最小化により現在のカメラの位置姿勢（6DOF）を求める．  
  
こちらもDirect methodの一般的な非線形最適化を用いている．  
モデルから得られた仮想画像を\\(I_{v}\\)，仮想Inverse depth imageを\\(\xi_{v}\\)とすると
$$
F(\psi)
=
\frac{1}{2}\sum_{\mathbf{u} \in \Omega}\left(f_{\mathbf{u}}\left(\psi \right)\right)^2
$$

$$
f_{\mathbf{u}} \left(\psi \right) 
=  I_{l}\left(\pi(K T_{lv}(\psi) \pi^{-1}(\mathbf{u}, \xi_{v}(\mathbf{u})))\right) - I_v(\mathbf{u})
$$
  
ただし，\\(\psi \in \mathfrak{se}3\\)である．  
  
これらの計算をGPUで行うことでリアルタイム性を確保している．  
DTAMの利点の1つは，ある程度信頼できるDenseなモデルがある点である．  
これを活かし，輝度値の差を求める際にある閾値以上の点を棄却することで，障害物の影響を少なくしたロバストな位置姿勢推定が可能である．  
  
[1] Lovegrove, Steven, and Andrew J. Davison. "Real-time spherical mosaicing using whole image alignment." European Conference on Computer Vision. Springer, Berlin, Heidelberg, 2010.
