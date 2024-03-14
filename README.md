import java.io.*;
import javax.servlet.*;
import javax.servlet.http.*;
import java.sql.*;
import java.util.*;

public class PostServlet extends HttpServlet {
    private static final String JDBC_URL = "jdbc:mysql://localhost:3306/blog_db";
    private static final String JDBC_USER = "username";
    private static final String JDBC_PASSWORD = "password";

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String pathInfo = request.getPathInfo();
        if (pathInfo == null || pathInfo.equals("/")) {
            getAllPosts(request, response);
        } else {
            String[] parts = pathInfo.split("/");
            if (parts.length == 2 && parts[1].matches("\\d+")) {
                int postId = Integer.parseInt(parts[1]);
                getPostById(request, response, postId);
            } else {
                response.sendError(HttpServletResponse.SC_BAD_REQUEST);
            }
        }
    }

    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String title = request.getParameter("title");
        String content = request.getParameter("content");
        int userId = Integer.parseInt(request.getParameter("user_id"));

        addPost(title, content, userId);

        response.sendRedirect(request.getContextPath() + "/posts");
    }

    protected void doPut(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        int postId = Integer.parseInt(request.getParameter("id"));
        String title = request.getParameter("title");
        String content = request.getParameter("content");

        updatePost(postId, title, content);

        response.sendRedirect(request.getContextPath() + "/posts");
    }

    protected void doDelete(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        int postId = Integer.parseInt(request.getParameter("id"));

        deletePost(postId);

        response.sendRedirect(request.getContextPath() + "/posts");
    }

    private void getAllPosts(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        try (Connection conn = DriverManager.getConnection(JDBC_URL, JDBC_USER, JDBC_PASSWORD)) {
            PreparedStatement stmt = conn.prepareStatement("SELECT * FROM posts");
            ResultSet rs = stmt.executeQuery();
            List<Post> posts = new ArrayList<>();
            while (rs.next()) {
                Post post = new Post(rs.getInt("id"), rs.getString("title"), rs.getString("content"), rs.getTimestamp("created_at"));
                posts.add(post);
            }
            request.setAttribute("posts", posts);
            request.getRequestDispatcher("/index.jsp").forward(request, response);
        } catch (SQLException e) {
            e.printStackTrace();
            response.sendError(HttpServletResponse.SC_INTERNAL_SERVER_ERROR);
        }
    }

    private void getPostById(HttpServletRequest request, HttpServletResponse response, int postId) throws ServletException, IOException {
        try (Connection conn = DriverManager.getConnection(JDBC_URL, JDBC_USER, JDBC_PASSWORD)) {
            PreparedStatement stmt = conn.prepareStatement("SELECT * FROM posts WHERE id = ?");
            stmt.setInt(1, postId);
            ResultSet rs = stmt.executeQuery();
            if (rs.next()) {
                Post post = new Post(rs.getInt("id"), rs.getString("title"), rs.getString("content"), rs.getTimestamp("created_at"));
                request.setAttribute("post", post);
                request.getRequestDispatcher("/showPost.jsp").forward(request, response);
            } else {
                response.sendError(HttpServletResponse.SC_NOT_FOUND);
            }
        } catch (SQLException e) {
            e.printStackTrace();
            response.sendError(HttpServletResponse.SC_INTERNAL_SERVER_ERROR);
        }
    }

    private void addPost(String title, String content, int userId) {
        try (Connection conn = DriverManager.getConnection(JDBC_URL, JDBC_USER, JDBC_PASSWORD)) {
            PreparedStatement stmt = conn.prepareStatement("INSERT INTO posts (title, content, user_id) VALUES (?, ?, ?)");
            stmt.setString(1, title);
            stmt.setString(2, content);
            stmt.setInt(3, userId);
            stmt.executeUpdate();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    private void updatePost(int postId, String title, String content) {
        try (Connection conn = DriverManager.getConnection(JDBC_URL, JDBC_USER, JDBC_PASSWORD)) {
            PreparedStatement stmt = conn.prepareStatement("UPDATE posts SET title = ?, content = ? WHERE id = ?");
            stmt.setString(1, title);
            stmt.setString(2, content);
            stmt.setInt(3, postId);
            stmt.executeUpdate();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    private void deletePost(int postId) {
        try (Connection conn = DriverManager.getConnection(JDBC_URL, JDBC_USER, JDBC_PASSWORD)) {
            PreparedStatement stmt = conn.prepareStatement("DELETE FROM posts WHERE id = ?");
            stmt.setInt(1, postId);
            stmt.executeUpdate();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<html>
<head>
    <title>All Posts</title>
</head>
<body>
<h1>All Posts</h1>
<c:forEach var="post" items="${posts}">
    <div>
        <h2>${post.title}</h2>
        <p>${post.content}</p>
        <p>Created At: ${post.created_at}</p>
    </div>
</c:forEach>
</body>
</html>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Add Post</title>
</head>
<body>
<h1>Add Post</h1>
<form action="post" method="post">
    <label>Title: <input type="text" name="title"></label><br>
    <label>Content: <textarea name="content"></textarea></label><br>
    <input type="hidden" name="user_id" value="1"> <!-- Assuming user_id 1 is the default user -->
    <input type="submit" value="Add">
</form>
</body>
</html>
<%@
