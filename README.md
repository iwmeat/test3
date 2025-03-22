import java.io.FileWriter;
import java.io.IOException;
import java.io.PrintWriter;
//生成一个标准答案类
public class file2 {
    public static void main(String[] args) {
        // 定义文件路径
        String filePath = "answer.txt";

        try (PrintWriter writer = new PrintWriter(new FileWriter(filePath))) {
            // 生成1到100的数字并写入文件
            for (int i = 1; i <= 10; i++) {
                writer.println(i);
            }
            System.out.println("答案文件已生成: " + filePath);
        } catch (IOException e) {
            System.err.println("写入文件时发生错误: " + e.getMessage());
        }
    }
}
import java.io.FileWriter;
import java.io.IOException;
import java.util.Random;
import java.util.Stack;
import java.util.StringTokenizer;
//定义分数类
class Fraction {
    private int numerator;//分子
    private int denominator;//分母
//构造函数：处理约分和分母规范化
    public Fraction(int numerator, int denominator) {
        if (denominator == 0) throw new ArithmeticException("分母不能为零");
        //计算最大公约数进行约分
        int gcd = gcd(Math.abs(numerator), Math.abs(denominator));
        this.numerator = numerator / gcd;
        this.denominator = denominator / gcd;
        //确保分母始终为正
        if (this.denominator < 0) {
            this.numerator *= -1;
            this.denominator *= -1;
        }
    }
//分数加法：交叉相乘后相加
    public Fraction add(Fraction other) {
        int newNumerator = this.numerator * other.denominator +
                other.numerator * this.denominator;
        int newDenominator = this.denominator * other.denominator;
        return new Fraction(newNumerator, newDenominator);
    }
//分数减法：转换为加负数
    public Fraction subtract(Fraction other) {
        return this.add(new Fraction(-other.numerator, other.denominator));
    }
//分数乘法：分子乘分子，分母乘分母
    public Fraction multiply(Fraction other) {
        return new Fraction(
                this.numerator * other.numerator,
                this.denominator * other.denominator
        );
    }
//分数除法：转换为乘以倒数
    public Fraction divide(Fraction other) {
        return this.multiply(new Fraction(other.denominator, other.numerator));
    }
//欧几里得算法求最大公约数
    private int gcd(int a, int b) {
        return b == 0 ? a : gcd(b, a % b);
    }
//格式化输出：整数情况只显示分子
    @Override
    public String toString() {
        if (denominator == 1) return String.valueOf(numerator);
        return numerator + "/" + denominator;
    }
}

class FractionCalculator {
    //主计算方法：使用双栈处理表达式
    public static Fraction calculate(String expression) {
        //清理输入，移除空格和中文等号
        String cleanExp = expression.replaceAll("\\s+|＝", "");
        //使用StringTokenizer分割表达式元素
        StringTokenizer tokenizer = new StringTokenizer(cleanExp, "+-*/", true);
        Stack<Fraction> numbers = new Stack<>();
        Stack<String> operators = new Stack<>();
        //遍历所有token
        while (tokenizer.hasMoreTokens()) {
            String token = tokenizer.nextToken().trim();
            if (token.isEmpty()) continue;
            if (isNumber(token)) {
                //如果是数字或分数：解析后压入操作数栈
                numbers.push(parseFraction(token));
            } else if (isOperator(token)) {
                //运算符：比较优先级后处理栈内运算
                while (!operators.isEmpty() &&
                        precedence(operators.peek()) >= precedence(token)) {
                    processOperation(numbers, operators);
                }
                operators.push(token);
            }
        }
//处理剩余运算符
        while (!operators.isEmpty()) {
            processOperation(numbers, operators);
        }

        return numbers.pop();//最终结果
    }
//执行单次运算操作
    private static void processOperation(Stack<Fraction> nums, Stack<String> ops) {
        Fraction right = nums.pop();
        Fraction left = nums.pop();

        switch (ops.pop()) {
            case "+":
                nums.push(left.add(right));
                break;
            case "-":
                nums.push(left.subtract(right));
                break;
            case "*":
                nums.push(left.multiply(right));
                break;
            case "/":
                nums.push(left.divide(right));
                break;
        }
    }
//判断是否是数字或分数形式
    private static boolean isNumber(String token) {
        return token.matches("\\d+/\\d+|\\d+");
    }
//解析分数或整数
    private static Fraction parseFraction(String token) {
        if (token.contains("/")) {
            String[] parts = token.split("/");
            return new Fraction(Integer.parseInt(parts[0]), Integer.parseInt(parts[1]));
        }
        return new Fraction(Integer.parseInt(token), 1);
    }
//判断是否为运算符
    private static boolean isOperator(String token) {
        return "+-*/".contains(token) && token.length() == 1;
    }
//定义运算符优先级
    private static int precedence(String op) {
        return (op.equals("+") || op.equals("-")) ? 1 : 2;
    }
}

public class file {
    public static void main(String[] arg) {
        Random random = new Random();
        char[] operators = {'+', '-', '*', '/'};
        int numProblems = 10;//题目数量
        int max_number = 100;//最大数字
        String outputFilePath = "arithmetic_problems.txt";//题目文件
        String outputFilePath1 = "arithmetic_answers.txt";//答案文件
        try (FileWriter writer = new FileWriter(outputFilePath);
             FileWriter writer1 = new FileWriter(outputFilePath1)) {
            //循环生成题目
            for (int i = 1; i <= numProblems; i++) {
                int numOperators = random.nextInt(3) + 1;//随机生成运算符数量
                StringBuilder problem = new StringBuilder();

                // 生成第一个操作数
                generateOperand(problem, random, max_number, true);

                for (int j = 0; j < numOperators; j++) {
                    char operator = operators[random.nextInt(operators.length)];
                    problem.append(" ").append(operator).append(" ");
                    generateOperand(problem, random, max_number, operator == '/');
                }

                String expression = problem.toString();
                String problemLine = "题目" + i + ": " + expression + " = ";
                Fraction result = FractionCalculator.calculate(expression);
                String answerLine = ""  + result;

                writer.write(problemLine + "\n");
                writer1.write(answerLine + "\n");
            }
            System.out.println("题目和答案已保存到文件: " + outputFilePath+"与"+outputFilePath1);
        } catch (IOException | ArithmeticException e) {
            System.out.println("发生错误: " + e.getMessage());
            e.printStackTrace();
        }
    }

    private static void generateOperand(StringBuilder problem, Random random, int max_number, boolean isDivisor) {
        int num_choice = random.nextInt(2);
        if (num_choice == 1) {
            int num = generateNaturalNumber(random, max_number, isDivisor);
            problem.append(num);
        } else {
            generateProperFraction(problem, random, max_number);
        }
    }

    private static int generateNaturalNumber(Random random, int max_number, boolean isDivisor) {
        int num = random.nextInt(max_number);
        if (isDivisor && num == 0) {
            return random.nextInt(max_number - 1) + 1;
        }
        return num;
    }

    private static void generateProperFraction(StringBuilder problem, Random random, int max_number) {
        int numerator = random.nextInt(max_number - 1) + 1;
        int denominator = random.nextInt(max_number - numerator) + numerator + 1;
        problem.append(numerator).append("/").append(denominator);
    }
}
import java.io.*;
import java.util.ArrayList;
import java.util.List;
//对比解析标准答案与实际答案
public class file1 {

    // 对比两个答案文件并记录正确与错误的题号
    public static void compareAnswers(String expectedAnswersFile, String actualAnswersFile, String comparisonFile) {
        List<Integer> correctQuestions = new ArrayList<>(); // 存储正确的题号
        List<Integer> incorrectQuestions = new ArrayList<>(); // 存储错误的题号
        int corrnum=0;
        int incorrnum=0;
        try (BufferedReader expectedReader = new BufferedReader(new FileReader(expectedAnswersFile));
             BufferedReader actualReader = new BufferedReader(new FileReader(actualAnswersFile))) {

            String expectedLine;
            String actualLine;
            int lineNumber = 1;

            // 逐行对比两个文件
            while ((expectedLine = expectedReader.readLine()) != null && (actualLine = actualReader.readLine()) != null) {
                expectedLine = expectedLine.trim();
                actualLine = actualLine.trim();

                if (expectedLine.equals(actualLine)) {
                    correctQuestions.add(lineNumber);// 答案正确
                    corrnum++;
                } else {
                    incorrectQuestions.add(lineNumber); // 答案错误
                    incorrnum++;
                }
                lineNumber++;
            }

            // 生成对比结果文件
            try (BufferedWriter writer = new BufferedWriter(new FileWriter(comparisonFile))) {
                writer.write("correct Questions: " + corrnum + correctQuestions + "\n");
                writer.write("Incorrect Questions: " + incorrnum + incorrectQuestions + "\n");
            }

            System.out.println("Comparison results have been saved to " + comparisonFile);

        } catch (IOException e) {
            System.err.println("Error reading or writing file: " + e.getMessage());
        }
    }

    public static void main(String[] args) {
        // 文件路径

        String expectedAnswersFile = "answer.txt"; // 标准答案文件
        String actualAnswersFile = "arithmetic_answers.txt"; // 实际答案文件
        String comparisonFile = "comparison_results.txt"; // 对比结果文件

        // 对比答案并生成结果
        compareAnswers(expectedAnswersFile, actualAnswersFile, comparisonFile);
    }
}
