# Đặt phiên bản tối thiểu của CMake yêu cầu
cmake_minimum_required(VERSION 3.10)

# Định nghĩa macro in ra thông tin hợp lệ hoặc thành công
macro(print_complete_message message)
        execute_process(COMMAND ${CMAKE_COMMAND} -E env CLICOLOR_FORCE=1
                                ${CMAKE_COMMAND} -E cmake_echo_color  --green --bold "${message}")
endmacro()

# Định nghĩa macro in ra thông tin thông thường
macro(print_infor_message message)
        execute_process(COMMAND ${CMAKE_COMMAND} -E env CLICOLOR_FORCE=1
                                ${CMAKE_COMMAND} -E cmake_echo_color --blue --bold "${message}")
endmacro()

# Định nghĩa macro in ra thông tin khi bị lỗi
macro(print_error_message message)
        execute_process(COMMAND ${CMAKE_COMMAND} -E env CLICOLOR_FORCE=1
                                ${CMAKE_COMMAND} -E cmake_echo_color --red --bold "${message}")
endmacro()

# Định nghĩa macro in ra thông tin cảnh báo
macro(print_warning_message message)
        execute_process(COMMAND ${CMAKE_COMMAND} -E env CLICOLOR_FORCE=1
                                ${CMAKE_COMMAND} -E cmake_echo_color --yellow --bold "${message}")
endmacro()


# Định nghĩa màu sắc in ra màn hình khi bắt đầu chạy CMake
print_complete_message("--------------------------------------------- CMAKE STARTED RUN ---------------------------------------------")

# Đặt tên cho project
project(MyCProject)

# Đặt tiêu chuẩn C thành C99, yêu cầu bắt buộc hỗ trợ và không sử dụng các mở rộng
set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED True) # Cho biết liệu tiêu chuẩn C được chỉ định bởi CMAKE_C_STANDARD có bắt buộc phải tuân thủ hay không.
set(CMAKE_C_EXTENSIONS False)       # CMake sẽ yêu cầu trình biên dịch chỉ sử dụng tiêu chuẩn C thuần túy, không có các phần mở rộng, ví dụ -std=c99 thay vì -std=gnu99

# Nếu bạn muốn đảm bảo rằng cờ "-g" luôn được thêm vào bất kể CMake sử dụng trình biên dịch nào, bạn có thể dùng lệnh này. 
# Tuy nhiên, đây không phải là yêu cầu bắt buộc khi bạn đã thiết lập CMAKE_BUILD_TYPE thành Debug
set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -g")

# Phải có dòng này mới debug GDB bằng lệnh layout src được
set(CMAKE_BUILD_TYPE Debug)

# Thêm các thư mục chứa các file header cho toàn bộ dự án, lệnh này không cần target
include_directories(funtion_1)
include_directories(funtion_1/Header_Inside1)
include_directories(Lib_funtion)

# Thêm các thư mục con và xử lí các cấu trúc CMakeList bên trong thư mục con đó, thường phải đi kèm với include_directories hoặc target_include_directories, nếu không các thư viện hoặc executable được tạo ra không để làm gì cả   
add_subdirectory(funtion_1)

# Biến chứa tên các file source C ở thư mục hiện hành
set(LIST_NAME_SOURCE_C "")

# Thu thập tất cả các tệp .c trong thư mục hiện tại không bao gồm các thư mục con (nếu muốn tìm trong các thư mục con dùng GLOB_RECURSE)
file(GLOB PATH_SOURCE_FILES "${CMAKE_SOURCE_DIR}/*.c")

# Lặp lại qua từng tệp .c để lấy tên của các file source ở thư mục hiện tại
foreach(PATH_SOURCE ${PATH_SOURCE_FILES})
    # Lấy tên tệp có phần mở rộng với keyword là NAME (ví dụ: main.c hoặc foo.c)
    get_filename_component(FILENAME ${PATH_SOURCE} NAME)
    # Thêm tên file source vào danh sách
    list(APPEND LIST_NAME_SOURCE_C ${FILENAME})
    # Hiện thông tin tên file tìm thấy
    print_infor_message("[CMakeList_In_CMAKE_GENERATE_PROJECT]: Found Sources Code ---> ${FILENAME}")
endforeach()

# Thêm tệp nguồn vào project
add_executable(MyCProject ${LIST_NAME_SOURCE_C})

# 
target_link_libraries(MyCProject PRIVATE Lib_function_1)
# target_include_directories(MyCProject PUBLIC Lib_funtion)

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
    add_dependencies(MyCProject ${FILENAME_WE}_preprocess)
endforeach()

# (Tùy chọn) Thêm các tùy chọn biên dịch đặc biệt
# target_compile_options(MyCProject PRIVATE -Wall -Wextra -O2)

# set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -maccumulate-outgoing-args")
# hoặc
# set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -maccumulate-outgoing-args")

# if(CMAKE_SIZEOF_VOID_P EQUAL 8)
#     set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -m64")
#     # hoặc
#     set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -m64")
# else()
#     set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -m32")
#     # hoặc
#     set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -m32")
# endif()

