#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <ctype.h>

#define MAX_STUDENTS 100
#define MAX_NAME_LENGTH 50
#define MAX_COURSES 10
#define FILENAME "students.dat"

typedef struct {
    char student_id[20];
    char name[MAX_NAME_LENGTH];
    int age;
    float scores[MAX_COURSES];
} Student;

typedef struct {
    Student students[MAX_STUDENTS];
    int count;
    char course_names[MAX_COURSES][50];
    int course_count;
} StudentManager;


void init_manager(StudentManager *manager);
int add_student(StudentManager *manager, const char *id, const char *name, int age, const float *scores);
int delete_student(StudentManager *manager, const char *id);
int update_student(StudentManager *manager, const char *id, const char *name, int age, const float *scores);
Student *query_student(StudentManager *manager, const char *id);
void sort_by_score(StudentManager *manager, int course_index, int ascending);
void display_student(const Student *student, const StudentManager *manager);
void display_all_students(const StudentManager *manager);
int save_to_file(const StudentManager *manager);
int load_from_file(StudentManager *manager);
void clear_input_buffer();

int main() {
    StudentManager manager;
    init_manager(&manager);
    
    
    if (load_from_file(&manager) != 0) {
        printf("警告: 加载数据失败，将使用空数据\n");
    }
    

    if (manager.course_count < MAX_COURSES) {
        strcpy(manager.course_names[manager.course_count++], "英语");
        strcpy(manager.course_names[manager.course_count++], "计算机");
    }
    
    int choice;
    char id[20];
    char name[MAX_NAME_LENGTH];
    int age;
    float scores[MAX_COURSES];
    Student *student;
    
    while (1) {
        printf("\n学生信息管理系统\n");
        printf("1. 添加学生\n");
        printf("2. 删除学生\n");
        printf("3. 修改信息\n");
        printf("4. 查询信息\n");
        printf("5. 成绩排序\n");
        printf("6. 统计人数\n");
        printf("7. 显示全部\n");
        printf("8. 存档退出\n");
        printf("请选择操作: ");
        
        if (scanf("%d", &choice) != 1) {
            printf("输入无效，请输入数字\n");
            clear_input_buffer();
            continue;
        }
        clear_input_buffer();
        
        switch (choice) {
            case 1:
                printf("请输入学号: ");
                fgets(id, sizeof(id), stdin);
                id[strcspn(id, "\n")] = 0;  
                
                printf("请输入姓名: ");
                fgets(name, sizeof(name), stdin);
                name[strcspn(name, "\n")] = 0;
                
                printf("请输入年龄: ");
                if (scanf("%d", &age) != 1) {
                    printf("年龄输入无效\n");
                    clear_input_buffer();
                    break;
                }
                clear_input_buffer();
                
                for (int i = 0; i < manager.course_count; i++) {
                    printf("请输入%s成绩: ", manager.course_names[i]);
                    if (scanf("%f", &scores[i]) != 1) {
                        printf("成绩输入无效\n");
                        clear_input_buffer();
                        break;
                    }
                    clear_input_buffer();
                }
                
                if (add_student(&manager, id, name, age, scores)) {
                    printf("添加成功！\n");
                    save_to_file(&manager);
                } else {
                    printf("添加失败：学号已存在\n");
                }
                break;
                
            case 2:
                printf("请输入要删除的学号: ");
                fgets(id, sizeof(id), stdin);
                id[strcspn(id, "\n")] = 0;
                
                if (delete_student(&manager, id)) {
                    printf("删除成功！\n");
                    save_to_file(&manager);
                } else {
                    printf("删除失败：未找到该学生\n");
                }
                break;
                
            case 3:
                printf("请输入要修改的学号: ");
                fgets(id, sizeof(id), stdin);
                id[strcspn(id, "\n")] = 0;
                
                student = query_student(&manager, id);
                if (!student) {
                    printf("未找到该学生\n");
                    break;
                }
                
                printf("请输入新姓名 (留空保持不变): ");
                fgets(name, sizeof(name), stdin);
                name[strcspn(name, "\n")] = 0;
                if (strlen(name) > 0) {
                    strcpy(student->name, name);
                }
                
                printf("请输入新年龄 (输入0保持不变): ");
                if (scanf("%d", &age) != 1) {
                    printf("年龄输入无效\n");
                    clear_input_buffer();
                    break;
                }
                clear_input_buffer();
                if (age > 0) {
                    student->age = age;
                }
                
                for (int i = 0; i < manager.course_count; i++) {
                    printf("请输入新%s成绩 (输入-1保持不变): ", manager.course_names[i]);
                    if (scanf("%f", &scores[i]) != 1) {
                        printf("成绩输入无效\n");
                        clear_input_buffer();
                        break;
                    }
                    clear_input_buffer();
                    if (scores[i] >= 0) {
                        student->scores[i] = scores[i];
                    }
                }
                
                printf("修改成功！\n");
                save_to_file(&manager);
                break;
                
            case 4:
                printf("请输入要查询的学号: ");
                fgets(id, sizeof(id), stdin);
                id[strcspn(id, "\n")] = 0;
                
                student = query_student(&manager, id);
                if (student) {
                    display_student(student, &manager);
                } else {
                    printf("未找到该学生\n");
                }
                break;
                
            case 5: {
                int course_index;
                printf("请选择排序课程:\n");
                for (int i = 0; i < manager.course_count; i++) {
                    printf("%d. %s\n", i + 1, manager.course_names[i]);
                }
                printf("请选择课程编号: ");
                if (scanf("%d", &course_index) != 1 || course_index < 1 || course_index > manager.course_count) {
                    printf("课程选择无效\n");
                    clear_input_buffer();
                    break;
                }
                clear_input_buffer();
                
                printf("排序方式:\n");
                printf("1. 升序\n");
                printf("2. 降序\n");
                printf("请选择排序方式: ");
                int sort_order;
                if (scanf("%d", &sort_order) != 1 || (sort_order != 1 && sort_order != 2)) {
                    printf("排序方式选择无效\n");
                    clear_input_buffer();
                    break;
                }
                clear_input_buffer();
                
                sort_by_score(&manager, course_index - 1, sort_order == 1);
                printf("%s成绩排名:\n", manager.course_names[course_index - 1]);
                display_all_students(&manager);
                break;
            }
                
            case 6:
                printf("学生总数: %d\n", manager.count);
                break;
                
            case 7:
                display_all_students(&manager);
                break;
                
            case 8:
                if (save_to_file(&manager) == 0) {
                    printf("数据已保存，感谢使用！\n");
                } else {
                    printf("保存数据失败！\n");
                }
                return 0;
                
            default:
                printf("无效选择，请重新输入！\n");
        }
    }
}

void init_manager(StudentManager *manager) {
    manager->count = 0;
    manager->course_count = 0;
}

int add_student(StudentManager *manager, const char *id, const char *name, int age, const float *scores) {
    if (manager->count >= MAX_STUDENTS) {
        return 0;  
    }
    

    for (int i = 0; i < manager->count; i++) {
        if (strcmp(manager->students[i].student_id, id) == 0) {
            return 0;  
        }
    }
    
  
    Student *new_student = &manager->students[manager->count++];
    strcpy(new_student->student_id, id);
    strcpy(new_student->name, name);
    new_student->age = age;
    
    for (int i = 0; i < manager->course_count; i++) {
        new_student->scores[i] = scores[i];
    }
    
    return 1;  
}

int delete_student(StudentManager *manager, const char *id) {
    for (int i = 0; i < manager->count; i++) {
        if (strcmp(manager->students[i].student_id, id) == 0) {
         
            for (int j = i; j < manager->count - 1; j++) {
                manager->students[j] = manager->students[j + 1];
            }
            manager->count--;
            return 1;  
        }
    }
    return 0;  
}

int update_student(StudentManager *manager, const char *id, const char *name, int age, const float *scores) {
    for (int i = 0; i < manager->count; i++) {
        if (strcmp(manager->students[i].student_id, id) == 0) {
            if (strlen(name) > 0) {
                strcpy(manager->students[i].name, name);
            }
            if (age > 0) {
                manager->students[i].age = age;
            }
            for (int j = 0; j < manager->course_count; j++) {
                if (scores[j] >= 0) {
                    manager->students[i].scores[j] = scores[j];
                }
            }
            return 1;  
        }
    }
    return 0; 
}

Student *query_student(StudentManager *manager, const char *id) {
    for (int i = 0; i < manager->count; i++) {
        if (strcmp(manager->students[i].student_id, id) == 0) {
            return &manager->students[i];
        }
    }
    return NULL;  
}

void sort_by_score(StudentManager *manager, int course_index, int ascending) {
 
    for (int i = 0; i < manager->count - 1; i++) {
        for (int j = 0; j < manager->count - i - 1; j++) {
            float score1 = manager->students[j].scores[course_index];
            float score2 = manager->students[j + 1].scores[course_index];
            
            if ((ascending && score1 > score2) || (!ascending && score1 < score2)) {
                
                Student temp = manager->students[j];
                manager->students[j] = manager->students[j + 1];
                manager->students[j + 1] = temp;
            }
        }
    }
}

void display_student(const Student *student, const StudentManager *manager) {
    if (!student) return;
    
    printf("学号: %s\n", student->student_id);
    printf("姓名: %s\n", student->name);
    printf("年龄: %d\n", student->age);
    printf("成绩:\n");
    
    for (int i = 0; i < manager->course_count; i++) {
        printf("  %s: %.1f\n", manager->course_names[i], student->scores[i]);
    }
    
    printf("--------------------\n");
}

void display_all_students(const StudentManager *manager) {
    if (manager->count == 0) {
        printf("暂无学生信息！\n");
        return;
    }
    
    for (int i = 0; i < manager->count; i++) {
        display_student(&manager->students[i], manager);
    }
}

int save_to_file(const StudentManager *manager) {
    FILE *file = fopen(FILENAME, "wb");
    if (!file) {
        printf("无法打开文件 %s\n", FILENAME);
        return 1;
    }

    fwrite(&manager->course_count, sizeof(int), 1, file);
    for (int i = 0; i < manager->course_count; i++) {
        fwrite(manager->course_names[i], sizeof(char), MAX_NAME_LENGTH, file);
    }

    fwrite(&manager->count, sizeof(int), 1, file);

    fwrite(manager->students, sizeof(Student), manager->count, file);
    
    fclose(file);
    return 0;
}

int load_from_file(StudentManager *manager) {
    FILE *file = fopen(FILENAME, "rb");
    if (!file) {
        return 1;  
    }

    fread(&manager->course_count, sizeof(int), 1, file);
    for (int i = 0; i < manager->course_count; i++) {
        fread(manager->course_names[i], sizeof(char), MAX_NAME_LENGTH, file);
    }
    

    fread(&manager->count, sizeof(int), 1, file);

    if (manager->count > 0) {
        fread(manager->students, sizeof(Student), manager->count, file);
    }
    
    fclose(file);
    return 0;
}

void clear_input_buffer() {
    int c;
    while ((c = getchar()) != '\n' && c != EOF);
