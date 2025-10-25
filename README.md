import java.sql.*;
import java.util.*;
public class StudentManagementApp {
    static class Student {
        int id;
        String name, dept;
        float marks;

        Student(int id, String name, String dept, float marks) {
            this.id = id;
            this.name = name;
            this.dept = dept;
            this.marks = marks;
        }

        @Override
        public String toString() {
            return String.format("%-5d %-15s %-15s %-5.2f", id, name, dept, marks);
        }
    }
    static class StudentDAO {
        final String URL = "jdbc:mysql://localhost:3306/studentdb";
        final String USER = "root";
        final String PASS = "root";

        StudentDAO() {
            try { Class.forName("com.mysql.cj.jdbc.Driver"); }
            catch (Exception e) { System.out.println("Driver load error: " + e); }
        }

        Connection getCon() throws SQLException {
            return DriverManager.getConnection(URL, USER, PASS);
        }

        boolean add(Student s) {
            String sql = "INSERT INTO students(name, department, marks) VALUES (?,?,?)";
            try (Connection c = getCon(); PreparedStatement ps = c.prepareStatement(sql)) {
                ps.setString(1, s.name); ps.setString(2, s.dept); ps.setFloat(3, s.marks);
                return ps.executeUpdate() > 0;
            } catch (Exception e) { System.out.println(e); return false; }
        }

        List<Student> getAll() {
            List<Student> list = new ArrayList<>();
            try (Connection c = getCon();
                 Statement st = c.createStatement();
                 ResultSet rs = st.executeQuery("SELECT * FROM students")) {
                while (rs.next())
                    list.add(new Student(rs.getInt(1), rs.getString(2),
                            rs.getString(3), rs.getFloat(4)));
            } catch (Exception e) { System.out.println(e); }
            return list;
        }

        boolean update(Student s) {
            String sql = "UPDATE students SET name=?, department=?, marks=? WHERE student_id=?";
            try (Connection c = getCon(); PreparedStatement ps = c.prepareStatement(sql)) {
                ps.setString(1, s.name); ps.setString(2, s.dept);
                ps.setFloat(3, s.marks); ps.setInt(4, s.id);
                return ps.executeUpdate() > 0;
            } catch (Exception e) { System.out.println(e); return false; }
        }

        boolean delete(int id) {
            try (Connection c = getCon();
                 PreparedStatement ps = c.prepareStatement("DELETE FROM students WHERE student_id=?")) {
                ps.setInt(1, id);
                return ps.executeUpdate() > 0;
            } catch (Exception e) { System.out.println(e); return false; }
        }
    }

    // ===== VIEW / MENU =====
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        StudentDAO dao = new StudentDAO();

        while (true) {
            System.out.println("\n===== STUDENT MANAGEMENT =====");
            System.out.println("1. Add Student");
            System.out.println("2. View All Students");
            System.out.println("3. Update Student");
            System.out.println("4. Delete Student");
            System.out.println("5. Exit");
            System.out.print("Choose: ");
            int ch = sc.nextInt(); sc.nextLine();

            switch (ch) {
                case 1 -> {
                    System.out.print("Name: "); String n = sc.nextLine();
                    System.out.print("Dept: "); String d = sc.nextLine();
                    System.out.print("Marks: "); float m = sc.nextFloat();
                    if (dao.add(new Student(0, n, d, m))) System.out.println(" Added!");
                    else System.out.println(" Failed.");
                }
                case 2 -> {
                    System.out.printf("%-5s %-15s %-15s %-5s%n", "ID", "Name", "Dept", "Marks");
                    System.out.println("------------------------------------------------");
                    dao.getAll().forEach(System.out::println);
                }
                case 3 -> {
                    System.out.print("Enter ID: "); int id = sc.nextInt(); sc.nextLine();
                    System.out.print("New Name: "); String n = sc.nextLine();
                    System.out.print("New Dept: "); String d = sc.nextLine();
                    System.out.print("New Marks: "); float m = sc.nextFloat();
                    if (dao.update(new Student(id, n, d, m))) System.out.println(" Updated!");
                    else System.out.println("Failed.");
                }
                case 4 -> {
                    System.out.print("Enter ID to delete: "); int id = sc.nextInt();
                    if (dao.delete(id)) System.out.println("Deleted!");
                    else System.out.println(" Failed.");
                }
                case 5 -> { System.out.println("Bye!"); return; }
                default -> System.out.println("Invalid option!");
            }
        }
    }
}
