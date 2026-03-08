cmake_minimum_required(VERSION 3.0.2)
project(parallax_controleur)

find_package(catkin REQUIRED COMPONENTS
  geometry_msgs
  nav_msgs
  roscpp
  rospy
  sensor_msgs
  std_msgs
  tf
  tf2
  tf2_ros
)

# Rendre libgpiod optionnelle
find_library(GPIOD_LIBRARY
  NAMES gpiod
  PATHS /usr/lib /usr/local/lib /usr/lib/aarch64-linux-gnu /usr/lib/arm-linux-gnueabihf
)

if(GPIOD_LIBRARY)
  message(STATUS "✅ libgpiod trouvée (${GPIOD_LIBRARY}) - Compilation avec support GPIO réel")
  set(HAVE_GPIOD 1)
  
  # Récupérer aussi le répertoire d'inclusion
  find_path(GPIOD_INCLUDE_DIR
    NAMES gpiod.h
    PATHS /usr/include /usr/local/include
  )
  if(GPIOD_INCLUDE_DIR)
    message(STATUS "✅ En-têtes libgpiod trouvés dans ${GPIOD_INCLUDE_DIR}")
  else()
    message(WARNING "⚠️  Bibliothèque gpiod trouvée mais pas les en-têtes - compilation risquée")
  endif()
else()
  message(STATUS "ℹ️  libgpiod non trouvée - Compilation en mode simulation")
  set(HAVE_GPIOD 0)
  set(GPIOD_LIBRARY "")
  set(GPIOD_INCLUDE_DIR "")
endif()

catkin_package(
)

# Définir le préprocesseur pour le code C++
add_definitions(-DHAVE_GPIOD=${HAVE_GPIOD})

include_directories(
  ${catkin_INCLUDE_DIRS}
  ${GPIOD_INCLUDE_DIR}
  include
)

# ------------------------------------------------------------
# EXÉCUTABLE : controleur
# ------------------------------------------------------------
add_executable(controleur src/controleur.cpp)
target_link_libraries(controleur 
  ${catkin_LIBRARIES}
  ${GPIOD_LIBRARY}  # Sera vide si non trouvée, ce qui est acceptable
)

# ------------------------------------------------------------
# EXÉCUTABLE : obstacles_detecteur
# ------------------------------------------------------------  
add_executable(obstacles_detecteur src/obstacles_detecteur.cpp)    
target_include_directories(obstacles_detecteur PUBLIC 
  ${catkin_INCLUDE_DIRS}
  ${GPIOD_INCLUDE_DIR}
)
target_link_libraries(obstacles_detecteur 
  ${catkin_LIBRARIES}
  ${GPIOD_LIBRARY}  # Sera vide si non trouvée
)

# ------------------------------------------------------------
# EXÉCUTABLE : configurateur
# ------------------------------------------------------------
add_executable(configurateur src/configurateur.cpp)
target_link_libraries(configurateur 
  ${catkin_LIBRARIES}
  ${GPIOD_LIBRARY}  # Au cas où configurateur utiliserait aussi GPIO
)
add_dependencies(configurateur ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})

# ------------------------------------------------------------
# EXÉCUTABLE : afficheur
# ------------------------------------------------------------
add_executable(afficheur src/afficheur.cpp)
target_link_libraries(afficheur 
  ${catkin_LIBRARIES}
  ${GPIOD_LIBRARY}  # Au cas où afficheur utiliserait aussi GPIO
)