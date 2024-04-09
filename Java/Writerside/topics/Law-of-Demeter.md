# 六大原则之迪米特法则

迪米特法则（Law of Demeter，LoD）是美国 Northeastern University 的 Ian Holland 提出的，后来被 UML 的创始人之一 Booch 等人普及。因为经典的 《The Pragmatic Programmer》而广为人知。但它还有一个好记又好理解的名字，**最少知道原则（Least Knowledge Principle，LKP）**。

我们来写一个简单的例子, 首先创建一个 Student 类，其包含一个 `grade` 总分的属性：

```java
public class Student {
    private double grade;
    // ...getter/setter
}
```

然后来写一个 Teacher 的类，其包含一个获取班级学生列表的的方法:

```java
public class Teacher {
    public List<Student> getStudents() {}
}
```

接着有一个 Principal （校长）的类，他需要知道某个老师的班级所有学生的总分:

```java
public class Principal {
    public void query(Teacher teacher) {
        List<Student> students = teacher.getStudents();
        // 遍历 students, 调用 student.getGrade() 方法自行统计总分
    }
}
```

这就是一个违反了 DoL 的例子，校长根本没有必要知道每个学生的细节（`student.getGrade()`），应该由老师统计好然后返回给校长即可。所以，`Teacher` 类提供一个 `getStudentsTotalGrade()`的方法即可。

```java
public class Teacher {
    private List<Student> students;

    public Teacher(List<Student> students) {
        this.students = students;
    }

    public double getStudentsTotalGrade() {
        return students.stream().mapToDouble(Student::getGrade).sum();
    }
}

public class Principal {
    public void query(Teacher teacher) {
        double totalGrade = teacher.getStudentsTotalGrade();
        // 使用 totalGrade
    }
}
```

迪米特法则的应用范围确实远不止于代码层面的对象或类之间的交互，它同样适用于函数、模块、服务、甚至是系统级别的设计和架构。在所有这些层面，遵循迪米特法则都有助于减少不必要的耦合，增强系统的独立性和可维护性。

比如在微服务架构或分布式系统设计中，迪米特法则的应用有助于设计松耦合的服务，每个服务应该只依赖于其他服务提供的API接口，而不是服务的内部实现细节。这样的设计使得各个服务可以独立开发、部署和升级，增强了系统的灵活性和可扩展性。

虽然迪米特法则提供了减少耦合、提高系统独立性的指导原则，但在实际应用中也需要根据具体情况做出权衡。完全遵守迪米特法则可能会增加系统的复杂性，比如可能需要引入更多的中间层或代理来减少直接交互。因此，设计时应该在减少耦合和保持系统简洁之间寻找平衡点，确保既遵循了迪米特法则的精神，又没有过度设计。