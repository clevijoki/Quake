cmake_minimum_required(VERSION 2.8)
project(vkQuake C)

add_definitions(-D_CRT_SECURE_NO_WARNINGS)

add_subdirectory(SDL2)

add_definitions(-DPARANOID)

add_subdirectory(WinQuake)

