# include_directories(Header_Inside1)
# Thu thập tất cả các tệp .c trong thư mục hiện tại không bao gồm các thư mục con (nếu muốn tìm trong các thư mục con dùng GLOB_RECURSE)
file(GLOB PATH_SOURCE_FILES "${CMAKE_CURRENT_SOURCE_DIR}/*.c")

# Tạo thư viện bằng các mã nguồn bên trong thư mục funtion_1
add_library(Lib_function_1 function_1.c function_1.h)

# target_include_directories(Lib_function_1 PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
# target_include_directories(Lib_function_1 PUBLIC ${CMAKE_SOURCE_DIR}/funtion_1/Header_Inside1)

# Lặp lại qua từng tệp .c để tạo tệp .i tương ứng
foreach(SOURCE_FILE ${PATH_SOURCE_FILES})
    # Lấy tên tệp không có phần mở rộng với keyword NAME_WE (ví dụ: main hoặc foo)
    get_filename_component(FILENAME_WE ${SOURCE_FILE} NAME_WE)
    
    # Gọi add_custom_command cho từng tệp nguồn, add_custom_command sẽ add các lệnh custom để xử lí cho từng tệp tương ứng
    add_custom_command(
        OUTPUT ${CMAKE_BINARY_DIR}/${FILENAME_WE}.i
        COMMAND ${CMAKE_CXX_COMPILER} -E ${SOURCE_FILE} -o ${CMAKE_BINARY_DIR}/${FILENAME_WE}.i -I${CMAKE_SOURCE_DIR}/funtion_1 -I${CMAKE_SOURCE_DIR}/funtion_1/Header_Inside1 -I${CMAKE_SOURCE_DIR}/Lib_funtion
        DEPENDS ${SOURCE_FILE}
        COMMENT "Generating preprocessed file ${FILENAME_WE}.i"
    )
    
    # Tạo target tùy chỉnh cho mỗi file source, được hiểu là để xuất file .i cho từng file source
    add_custom_target(
        ${FILENAME_WE}_preprocess ALL
        DEPENDS ${CMAKE_BINARY_DIR}/${FILENAME_WE}.i
    )

    # Đảm bảo rằng quá trình build của MyCProject sẽ phụ thuộc vào việc tạo các file .i, do đó "quá trình tiền xử lý" sẽ được thực hiện "trước" khi biên dịch
    add_dependencies(Lib_function_1 ${FILENAME_WE}_preprocess)
endforeach()