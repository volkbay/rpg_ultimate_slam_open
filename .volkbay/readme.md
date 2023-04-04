# USLAM Installation Updates (2023, @volkbay) #
Ultimate SLAM denilen 2018 paperının açık kodları burada. Real-time veya ROSbag ile denemek için kullanabiliriz. ROS sistemi üzerinden çalışıyor bu sebeple Dockerfile yazdım bir tane. ROSun rviz diye bir gösterim toolunu kullanarak 3 boyutlu realtime gösteriyor.

> Tüm bilgiler GitHub ana page readme kısmında var.

1. Docker build ile kurulum yap.
2. Kalibr ile IMU calib yap. Bunun için bizim GH versiyonun kullanabilirsin. Bu aşamada timeshift parametresini almayı ve çıkan sonucu .swe manuel olarak çevirmeyi unutma. Maalesef *kalibr_swe_config* diye bir şey bulamadım.
3. Örnek bag indirip veya direkt real-time olarak denemek için Docker image run ederiz. Klasör yapılarına dikkat et.

    `docker run -it --rm --privileged --net=host -v /run/user/1000/gdm/Xauthority:/root/.Xauthority -v /tmp/.X11-unix/:/tmp/.X11-unix/ -v ${HOME}/prj/Ebv-uslam/dat/:/uslam_ws/src/rpg_ultimate_slam_open/data -v /dev/bus/usb:/dev/bus/usb -e DISPLAY --env=NVIDIA_DRIVER_CAPABILITIES=all --gpus all uslam`

    > Nvidia container toolkit kesinlikle gerekli. Ve şu parametreleri docker run sırasında girmeliyiz *.bashrc* yazmak çalışmıyor.

4. ROSun tanınması için her terminal açılışında:
  
    `source uslam_ws/devel/setup.bash`

5. Örnek veya bizim bagleri çalıştırmak için aşağıdaki gibi frameli-framesiz çalıştır.

    `roslaunch ze_vio_ceres ijrr17.launch bag_filename:=dynamic_6dof.bag`

    `roslaunch ze_vio_ceres ijrr17_events_only.launch bag_filename:=dynamic_6dof.bag`

## Real-Time Demo ##
Bunu yapabilmek yine calib edilmiş kamerayı bağladık ve yukarıdaki gibi Docker run edildi. Ek olarak container içinde driver yüklemeliyiz.

### Install Driver ###
Ayrıntılı bilgi [driver sayfasında](https://github.com/uzh-rpg/rpg_dvs_ros#driver-installation) ancak kısaca aşağı işlemleri yapmak gerekiyor. Bu kısımlar Dockerfile da yok (TODO).

`apt-get install software-properties-common`

`add-apt-repository ppa:ubuntu-toolchain-r/test`

`add-apt-repository ppa:inivation-ppa/inivation-bionic`

`apt update`

`apt-get install libcaer-dev`

`cd uslam_ws/src/`

`catkin build davis_ros_driver`

### Test Driver ###
Burada driver kameramızla test ediyoruz. Özellikle sondaki komut driver propertyleri için oldukça kullanışlı.

`catkin build dvs_renderer`

`roslaunch dvs_renderer davis_color.launch`

`rosrun rqt_reconfigure rqt_reconfigure`

> Bu aşamada dilersek `docker commit` alalım.

### Live Demo ###
Öncelikle calib dosyasının doğru yerde olması gerekiyor.

`cp uslam_ws/src/rpg_ultimate_slam_open/data/davis_calib_swe.yml uslam_ws/src/rpg_ultimate_slam_open/calibration/davis_calib_swe.yaml`

Ardında 3 terminal kullanarak şunları yapacağız. Bunun için Docker exec kullanıp source ile env tanıtmayı unutma. Timeshift param calibrasyondan geldi. Bir Rviz ile gösterim yapması beklenir.

  1. `roscore`
  2. `rosrun davis_ros_driver davis_ros_driver`
  3. `roslaunch ze_vio_ceres live_DAVIS240C.launch camera_name:=davis_calib_swe timeshift_cam_imu:=0.019081827388163747`

      veya
      
      `roslaunch ze_vio_ceres live_DAVIS240C_events_only.launch camera_name:=davis_calib_swe timeshift_cam_imu:=0.019081827388163747`

> Başlangıçta kameranın rest olması, iyi bir calib ve param tuning başarıyı arttırır. Param için GH'de ayrı bir sayfa ayırmışlar, onlar dışında çok fazla param var (terminalden bakabilirsin). Yine de çok driftli sonuçlar aldık.