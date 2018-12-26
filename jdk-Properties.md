```java 

package java.util;

import java.io.IOException;
import java.io.PrintStream;
import java.io.PrintWriter;
import java.io.InputStream;
import java.io.OutputStream;
import java.io.Reader;
import java.io.Writer;
import java.io.OutputStreamWriter;
import java.io.BufferedWriter;
import java.security.AccessController;
import java.security.PrivilegedAction;

import sun.util.spi.XmlPropertiesProvider;

/**
 * {@code Properties}类表示一组持久的属性。 {@code Properties}可以保存
 * 到流中或从流中加载。 属性列表中的每个键及其对应的值都是一个字符串。
 *  
 * 属性列表可以包含另一个属性列表作为其“默认值”; 
 * 如果在原始属性列表中找不到属性键，则搜索第二个属性列表。
 *  
 * 由于{@code Properties}继承自{@code Hashtable}，因此{@code put}和
 * {@code putAll}方法可应用于{@code Properties}对象。 
 * 强烈建议不要使用它们，因为它们允许调用者插入其键或值不是{@code Strings}
 * 的条目。 应该使用{@code setProperty}方法。 如果在包含非{@ code String}
 * 键或值的“受损”{@code Properties}对象上调用{@code store}或{@code save}方
 * 法，则调用将失败。 同样，如果在包含非{@ code String}键的“受损”
 * {@code Properties}对象上调用{@code propertyNames}或{@code list}方法，
 * 则调用将失败。
 *
 *  
 * {@link #load（java.io.Reader）load（Reader）}
 *  {@ link #store（java.io.Writer，java.lang.String）store（Writer，String）}
 * 方法从中加载和存储属性 基于字符的流以下面指定的简单的面向行的格式。
 *
 * {@link #load（java.io.InputStream）load（InputStream）}
 * {@ link #store（java.io.OutputStream，java.lang.String）store（OutputStream，String）}
 * 方法的工作方式与 load（Reader）/ store（Writer，String）对，输入/输出流除
 * 以ISO 8859-1字符编码编码。 无法在此编码中直接表示的字符可以使用
 * “Java＆trade”第3.3节中定义的Unicode转义来编写; 语言规范; 在转义序列中只允许一个
 * 'u'字符。 native2ascii工具可用于将属性文件转换为其他字符编码或从其他字符编码转换。
 *
 * {@link #loadFromXML（InputStream）}和{@link #storeToXML（OutputStream，String，String）}
 * 方法以简单的XML格式加载和存储属性。 默认情况下，使用UTF-8字符编码，但是如果需要，
 * 可以指定特定的编码。 实现需要支持UTF-8和UTF-16，并且可能支持其他编码。
 * XML属性文档具有以下DOCTYPE声明：
 *
 * <pre>
 * &lt;!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd"&gt;
 * </pre>
 * 请注意，导出或导入属性时不会访问系统URI（http://java.sun.com/dtd/properties.dtd）; 
 * 它只是作为一个字符串来唯一标识DTD，它是：
 * <pre>
 *    &lt;?xml version="1.0" encoding="UTF-8"?&gt;
 *
 *    &lt;!-- DTD for properties --&gt;
 *
 *    &lt;!ELEMENT properties ( comment?, entry* ) &gt;
 *
 *    &lt;!ATTLIST properties version CDATA #FIXED "1.0"&gt;
 *
 *    &lt;!ELEMENT comment (#PCDATA) &gt;
 *
 *    &lt;!ELEMENT entry (#PCDATA) &gt;
 *
 *    &lt;!ATTLIST entry key CDATA #REQUIRED&gt;
 * </pre>
 *
 * 此类是线程安全的：多个线程可以共享单个Properties对象，而无需外部同步。
 *
 * @see <a href="../../../technotes/tools/solaris/native2ascii.html">native2ascii tool for Solaris</a>
 * @see <a href="../../../technotes/tools/windows/native2ascii.html">native2ascii tool for Windows</a>
 *
 * @author  Arthur van Hoff
 * @author  Michael McCloskey
 * @author  Xueming Shen
 * @since   JDK1.0
 */
public
class Properties extends Hashtable<Object,Object> {
    /**
     * use serialVersionUID from JDK 1.1.X for interoperability
     */
     private static final long serialVersionUID = 4112578634029874840L;

    /**
     * 一个属性列表，其中包含此属性列表中未找到的任何键的默认值。
     *
     * @serial
     */
    protected Properties defaults;

    /**
     * 创建一个没有默认值的空属性列表。
     */
    public Properties() {
        this(null);
    }

    /**
     * 创建具有指定默认值的空属性列表。
     *
     * @param   defaults   the defaults.
     */
    public Properties(Properties defaults) {
        this.defaults = defaults;
    }

    /**
     * 调用Hashtable方法{@code put}。 提供与getProperty方法的并行性。 
     * 强制使用字符串作为属性键和值。 返回的值是对{@code put}的Hashtable调用的结果。
     *
     * @param key the key to be placed into this property list.
     * @param value the value corresponding to <tt>key</tt>.
     * @return     the previous value of the specified key in this property
     *             list, or {@code null} if it did not have one.
     * @see #getProperty
     * @since    1.2
     */
    public synchronized Object setProperty(String key, String value) {
        return put(key, value);
    }


    /**
     * 以简单的面向行的格式从输入字符流中读取属性列表（键和元素对）。
     * 
     * 属性按行处理。 有两种线，自然线和逻辑线。 
     * 自然线被定义为一行字符，它们由一组行终止符（{@code \ n}或{@code \ r}
     * 或{@code \ r \ n}）或结尾处终止。 流。 
     * 自然线可以是空行，注释行，也可以是全部或部分键元素对。 
     * 逻辑行保存键元素对的所有数据，通过使用反斜杠字符
     * {@code \}转义行终止符序列，可以将其分布在几个相邻的自然行中。 
     * 请注意，注释行不能以这种方式扩展; 作为评论的每条自然线必须有自己的评论指标，
     * 如下所述。 从输入读取行，直到到达流的末尾。
     *
     * 
     * 仅包含空格字符的自然行被视为空白，将被忽略。 注释行有一个ASCII {@code'＃'}
     * 或{@code'！'}作为其第一个非空格字符; 注释行也被忽略，不对关键元素信息进行编
     * 码。 除了行终止符，此格式还会考虑字符空格（{@code''}，{@ code'\ u005Cu0020'}），
     * 制表符（{@code'\ t'}，{@ code'\ u005Cu0009'}） 和表单Feed（{@code'\ f'}，
     * {@ code'\ u005Cu000C'}）为空格。
     *
     * 
     *  
    * 如果逻辑行分布在多个自然行中，则反转行终止符序列的反斜杠，行终止符
    * 序列以及后续行开头的任何空格都不会影响键或元素值。 关键和元素解析（加载时）
    * 的讨论的其余部分将假设构成键和元素的所有字符在删除行连续字符后出现在单个自
    * 然行上。 注意，仅检查行终止符序列之前的字符以确定行终止符是否被转义是不够的。 
    * 必须有奇数个连续的反斜杠才能对行终止符进行转义。 由于输入是从左到右处理的，
    * 因此在行终止符（或其他地方）之前的非零偶数个2n个连续反斜杠在转义处理之后编
    * 码n个反斜杠。
     *
     *  
     * 该键包含从第一个非空格字符开始的行中的所有字符，但不包括第一个未转义的
     * {@code'='}，{@ code'：'}或空白字符 除了行终止符。 
     * 所有这些密钥终止字符都可以通过使用前面的反斜杠字符转义它们来包含在密钥中; 
     * 例如，
     *
     * {@code \:\=}<p>
     *
     * 将是双字符键{@code“：=”}。 可以使用{@code \ r}和{@code \ n}转义序列包含行
     * 终止符。 跳过键后的任何空格; 如果密钥后的第一个非空白字符是{@code'='}或
     * {@code'：'}，则忽略该字符，并且也会跳过其后的任何空格字符。
     * 该行上的所有剩余字符都成为相关元素字符串的一部分; 
     * 如果没有剩余字符，则该元素为空字符串{@code“”}。 
     * 一旦识别出构成密钥和元素的原始字符序列，就如上所述执行转义处理。
     *
     *  
     * 例如，以下三行中的每一行都指定了键{@code“Truth”}和相关的元素值
     * {@code "Beauty"}:
     * <pre>
     * Truth = Beauty
     *  Truth:Beauty
     * Truth                    :Beauty
     * </pre>
     * 作为另一个示例，以下三行指定单个属性：
     * <pre>
     * fruits                           apple, banana, pear, \
     *                                  cantaloupe, watermelon, \
     *                                  kiwi, mango
     * 关键是{@code“fruits”}，相关元素是：“苹果，香蕉，梨，哈密瓜，西瓜，猕
     * 猴桃，芒果”请注意，每个{@code \}前面都会出现一个空格，以便在出现后出现
     * 空格 最后结果中的每个逗号; {@code \}，行终止符和延续行上的前导空格
     * 仅被丢弃，不会被一个或多个其他字符替换。
     *  
     * 作为第三个例子，该行：
     * <pre>cheeses
     * </pre>
     * 指定键是{@code“cheeses”}，关联的元素是空字符串{@code“”}。
     * <p>
     * <a name="unicodeescapes"></a>
     * 键和元素中的字符可以用类似于字符和字符串文字的转义序列表示（参见
     * The Java＆trade; Language Specification的3.3和3.10.6节）。
     *
     * 与用于字符和字符串的字符转义序列和Unicode转义的区别是：
     *
     * <ul>
     * <li> Octal escapes are not recognized.
     *
     * 字符序列{@code \ b}不代表退格符。
     *
     * 该方法不会将无效转义字符之前的反斜杠字符{@code \}视为错误; 反斜杠是
     * 默默地丢弃的。 例如，在Java字符串中，序列{@code“\ z”}将导致编译时错误。 
     * 相反，这种方法会默默地删除反斜杠。 因此，此方法将两个字符序列{@code“\ 
     * b”}视为等同于单个字符{@code'b'}。
     *
     * 单引号和双引号不需要转义; 
     * 但是，根据上面的规则，以反斜杠开头的单引号和双引号字符仍然分别产生单
     * 引号和双引号字符。
     *
     * Unicode转义序列中只允许使用单个“u”字符。
     *
     * </ul>
     * <p>
     * 此方法返回后，指定的流仍保持打开状态。
     *
     * @param   reader   the input character stream.
     * @throws  IOException  if an error occurred when reading from the
     *          input stream.
     * @throws  IllegalArgumentException if a malformed Unicode escape
     *          appears in the input.
     * @since   1.6
     */
    public synchronized void load(Reader reader) throws IOException {
        load0(new LineReader(reader));
    }

    /**
     * 从输入字节流中读取属性列表（键和元素对）。 输入流采用简单的面向行的格式，
     * 如{@link #load（java.io.Reader）load（Reader）}中所指定，并假设使用
     * ISO 8859-1字符编码; 即每个字节是一个Latin1字符。 
     * 不是Latin1中的字符，以及某些特殊字符，使用Unicode转义表示在键和元素中，
     * 如“Java＆trade”第3.3节中所定义; 语言规范。 此方法返回后，指定的流仍保持打开状态。
     *
     * @param      inStream   the input stream.
     * @exception  IOException  if an error occurred when reading from the
     *             input stream.
     * @throws     IllegalArgumentException if the input stream contains a
     *             malformed Unicode escape sequence.
     * @since 1.2
     */
    public synchronized void load(InputStream inStream) throws IOException {
        load0(new LineReader(inStream));
    }

    private void load0 (LineReader lr) throws IOException {
        char[] convtBuf = new char[1024];
        int limit;
        int keyLen;
        int valueStart;
        char c;
        boolean hasSep;
        boolean precedingBackslash;

        while ((limit = lr.readLine()) >= 0) {
            c = 0;
            keyLen = 0;
            valueStart = limit;
            hasSep = false;

            //System.out.println("line=<" + new String(lineBuf, 0, limit) + ">");
            precedingBackslash = false;
            while (keyLen < limit) {
                c = lr.lineBuf[keyLen];
                //need check if escaped.
                if ((c == '=' ||  c == ':') && !precedingBackslash) {
                    valueStart = keyLen + 1;
                    hasSep = true;
                    break;
                } else if ((c == ' ' || c == '\t' ||  c == '\f') && !precedingBackslash) {
                    valueStart = keyLen + 1;
                    break;
                }
                if (c == '\\') {
                    precedingBackslash = !precedingBackslash;
                } else {
                    precedingBackslash = false;
                }
                keyLen++;
            }
            while (valueStart < limit) {
                c = lr.lineBuf[valueStart];
                if (c != ' ' && c != '\t' &&  c != '\f') {
                    if (!hasSep && (c == '=' ||  c == ':')) {
                        hasSep = true;
                    } else {
                        break;
                    }
                }
                valueStart++;
            }
            String key = loadConvert(lr.lineBuf, 0, keyLen, convtBuf);
            String value = loadConvert(lr.lineBuf, valueStart, limit - valueStart, convtBuf);
            put(key, value);
        }
    }

    /* 从InputStream / Reader读入“逻辑行”，跳过所有注释和空白行，并从“自然行”
     * 的开头过滤掉那些前导空格字符（\ u0020，\ u0009和\ u000c）。
     * 方法返回“逻辑行”的字符长度，并将该行存储在“lineBuf”中。
     */
    class LineReader {
        public LineReader(InputStream inStream) {
            this.inStream = inStream;
            inByteBuf = new byte[8192];
        }

        public LineReader(Reader reader) {
            this.reader = reader;
            inCharBuf = new char[8192];
        }

        byte[] inByteBuf;
        char[] inCharBuf;
        char[] lineBuf = new char[1024];
        int inLimit = 0;
        int inOff = 0;
        InputStream inStream;
        Reader reader;

        int readLine() throws IOException {
            int len = 0;
            char c = 0;

            boolean skipWhiteSpace = true;
            boolean isCommentLine = false;
            boolean isNewLine = true;
            boolean appendedLineBegin = false;
            boolean precedingBackslash = false;
            boolean skipLF = false;

            while (true) {
                if (inOff >= inLimit) {
                    inLimit = (inStream==null)?reader.read(inCharBuf)
                                              :inStream.read(inByteBuf);
                    inOff = 0;
                    if (inLimit <= 0) {
                        if (len == 0 || isCommentLine) {
                            return -1;
                        }
                        if (precedingBackslash) {
                            len--;
                        }
                        return len;
                    }
                }
                if (inStream != null) {
                    //The line below is equivalent to calling a
                    //ISO8859-1 decoder.
                    c = (char) (0xff & inByteBuf[inOff++]);
                } else {
                    c = inCharBuf[inOff++];
                }
                if (skipLF) {
                    skipLF = false;
                    if (c == '\n') {
                        continue;
                    }
                }
                if (skipWhiteSpace) {
                    if (c == ' ' || c == '\t' || c == '\f') {
                        continue;
                    }
                    if (!appendedLineBegin && (c == '\r' || c == '\n')) {
                        continue;
                    }
                    skipWhiteSpace = false;
                    appendedLineBegin = false;
                }
                if (isNewLine) {
                    isNewLine = false;
                    if (c == '#' || c == '!') {
                        isCommentLine = true;
                        continue;
                    }
                }

                if (c != '\n' && c != '\r') {
                    lineBuf[len++] = c;
                    if (len == lineBuf.length) {
                        int newLength = lineBuf.length * 2;
                        if (newLength < 0) {
                            newLength = Integer.MAX_VALUE;
                        }
                        char[] buf = new char[newLength];
                        System.arraycopy(lineBuf, 0, buf, 0, lineBuf.length);
                        lineBuf = buf;
                    }
                    //flip the preceding backslash flag
                    if (c == '\\') {
                        precedingBackslash = !precedingBackslash;
                    } else {
                        precedingBackslash = false;
                    }
                }
                else {
                    // reached EOL
                    if (isCommentLine || len == 0) {
                        isCommentLine = false;
                        isNewLine = true;
                        skipWhiteSpace = true;
                        len = 0;
                        continue;
                    }
                    if (inOff >= inLimit) {
                        inLimit = (inStream==null)
                                  ?reader.read(inCharBuf)
                                  :inStream.read(inByteBuf);
                        inOff = 0;
                        if (inLimit <= 0) {
                            if (precedingBackslash) {
                                len--;
                            }
                            return len;
                        }
                    }
                    if (precedingBackslash) {
                        len -= 1;
                        //skip the leading whitespace characters in following line
                        skipWhiteSpace = true;
                        appendedLineBegin = true;
                        precedingBackslash = false;
                        if (c == '\r') {
                            skipLF = true;
                        }
                    } else {
                        return len;
                    }
                }
            }
        }
    }

    /*
     * Converts encoded &#92;uxxxx to unicode chars
     * and changes special saved chars to their original forms
     */
    private String loadConvert (char[] in, int off, int len, char[] convtBuf) {
        if (convtBuf.length < len) {
            int newLen = len * 2;
            if (newLen < 0) {
                newLen = Integer.MAX_VALUE;
            }
            convtBuf = new char[newLen];
        }
        char aChar;
        char[] out = convtBuf;
        int outLen = 0;
        int end = off + len;

        while (off < end) {
            aChar = in[off++];
            if (aChar == '\\') {
                aChar = in[off++];
                if(aChar == 'u') {
                    // Read the xxxx
                    int value=0;
                    for (int i=0; i<4; i++) {
                        aChar = in[off++];
                        switch (aChar) {
                          case '0': case '1': case '2': case '3': case '4':
                          case '5': case '6': case '7': case '8': case '9':
                             value = (value << 4) + aChar - '0';
                             break;
                          case 'a': case 'b': case 'c':
                          case 'd': case 'e': case 'f':
                             value = (value << 4) + 10 + aChar - 'a';
                             break;
                          case 'A': case 'B': case 'C':
                          case 'D': case 'E': case 'F':
                             value = (value << 4) + 10 + aChar - 'A';
                             break;
                          default:
                              throw new IllegalArgumentException(
                                           "Malformed \\uxxxx encoding.");
                        }
                     }
                    out[outLen++] = (char)value;
                } else {
                    if (aChar == 't') aChar = '\t';
                    else if (aChar == 'r') aChar = '\r';
                    else if (aChar == 'n') aChar = '\n';
                    else if (aChar == 'f') aChar = '\f';
                    out[outLen++] = aChar;
                }
            } else {
                out[outLen++] = aChar;
            }
        }
        return new String (out, 0, outLen);
    }

    /*
     * 将unicode转换为encode \ uxxxx并使用前面的斜杠转义特殊字符
     */
    private String saveConvert(String theString,
                               boolean escapeSpace,
                               boolean escapeUnicode) {
        int len = theString.length();
        int bufLen = len * 2;
        if (bufLen < 0) {
            bufLen = Integer.MAX_VALUE;
        }
        StringBuffer outBuffer = new StringBuffer(bufLen);

        for(int x=0; x<len; x++) {
            char aChar = theString.charAt(x);
            // Handle common case first, selecting largest block that
            // avoids the specials below
            if ((aChar > 61) && (aChar < 127)) {
                if (aChar == '\\') {
                    outBuffer.append('\\'); outBuffer.append('\\');
                    continue;
                }
                outBuffer.append(aChar);
                continue;
            }
            switch(aChar) {
                case ' ':
                    if (x == 0 || escapeSpace)
                        outBuffer.append('\\');
                    outBuffer.append(' ');
                    break;
                case '\t':outBuffer.append('\\'); outBuffer.append('t');
                          break;
                case '\n':outBuffer.append('\\'); outBuffer.append('n');
                          break;
                case '\r':outBuffer.append('\\'); outBuffer.append('r');
                          break;
                case '\f':outBuffer.append('\\'); outBuffer.append('f');
                          break;
                case '=': // Fall through
                case ':': // Fall through
                case '#': // Fall through
                case '!':
                    outBuffer.append('\\'); outBuffer.append(aChar);
                    break;
                default:
                    if (((aChar < 0x0020) || (aChar > 0x007e)) & escapeUnicode ) {
                        outBuffer.append('\\');
                        outBuffer.append('u');
                        outBuffer.append(toHex((aChar >> 12) & 0xF));
                        outBuffer.append(toHex((aChar >>  8) & 0xF));
                        outBuffer.append(toHex((aChar >>  4) & 0xF));
                        outBuffer.append(toHex( aChar        & 0xF));
                    } else {
                        outBuffer.append(aChar);
                    }
            }
        }
        return outBuffer.toString();
    }

    private static void writeComments(BufferedWriter bw, String comments)
        throws IOException {
        bw.write("#");
        int len = comments.length();
        int current = 0;
        int last = 0;
        char[] uu = new char[6];
        uu[0] = '\\';
        uu[1] = 'u';
        while (current < len) {
            char c = comments.charAt(current);
            if (c > '\u00ff' || c == '\n' || c == '\r') {
                if (last != current)
                    bw.write(comments.substring(last, current));
                if (c > '\u00ff') {
                    uu[2] = toHex((c >> 12) & 0xf);
                    uu[3] = toHex((c >>  8) & 0xf);
                    uu[4] = toHex((c >>  4) & 0xf);
                    uu[5] = toHex( c        & 0xf);
                    bw.write(new String(uu));
                } else {
                    bw.newLine();
                    if (c == '\r' &&
                        current != len - 1 &&
                        comments.charAt(current + 1) == '\n') {
                        current++;
                    }
                    if (current == len - 1 ||
                        (comments.charAt(current + 1) != '#' &&
                        comments.charAt(current + 1) != '!'))
                        bw.write("#");
                }
                last = current + 1;
            }
            current++;
        }
        if (last != current)
            bw.write(comments.substring(last, current));
        bw.newLine();
    }

    /**
     *  调用{@code store（OutputStream out，String comments）}方法并禁止抛出的IOExceptions。
     *
     * @deprecated如果在保存属性列表时发生I / O错误，则此方法不会抛出IOException。 
     * 保存属性列表的首选方法是通过{@code store（OutputStream out，String 
     * comments）}方法或{@code storeToXML（OutputStream os，String comment）}方法。
     *
     * @param   out      an output stream.
     * @param   comments   a description of the property list.
     * @exception  ClassCastException  if this {@code Properties} object
     *             contains any keys or values that are not
     *             {@code Strings}.
     */
    @Deprecated
    public void save(OutputStream out, String comments)  {
        try {
            store(out, comments);
        } catch (IOException e) {
        }
    }

    /**
     * 将此{@code Properties}表中的此属性列表（键和元素对）以适合使用
     * {@link #load（java.io.Reader）load（Reader）}方法的格式写入输出字符流。
     *  
     * 此{@code Properties}表的默认表中的属性（如果有）不是由此方法写出的。
     *  
     * 如果comments参数不为null，则首先将ASCII {@code＃}字符，注释字符串和行
     * 分隔符写入输出流。 因此，{@code comments}可以作为识别注释。 换行符
     * （'\ n'），回车符（'\ r'）或回车符后面的换行符中的任何一个都被
     * {@code Writer}生成的行分隔符替换为 如果注释中的下一个字符不是字符{@code＃}或
     * 字符{@code！}，则在该行分隔符之后写出ASCII {@code＃}。
     *  
     * 接下来，始终写入注释行，包括ASCII {@code＃}字符，当前日期和时间（就像当
     * 前时间{@code Date}的{@code toString}方法一样），以及 由{@code Writer}生成的行分隔符。
     *  
     * 然后写出此{@code Properties}表中的每个条目，每行一个。 
     * 对于每个条目，写入密钥字符串，然后写入ASCII {@code =}，然后写入关联的元素字符串。
     * 对于密钥，所有空格字符都使用前面的{@code \}字符编写。 
     * 对于元素，前导空格字符，但不是嵌入或尾随空格字符，使用前面的{@code \}字符编写。 
     * 密钥和元素字符{@code＃}，{@ code！}，{@ code =}和{@code：}
     * 都使用前面的反斜杠编写，以确保它们已正确加载。
     *  
     * 写入条目后，将刷新输出流。 此方法返回后，输出流保持打开状态。
     *  
     *
     * @param   writer      an output character stream writer.
     * @param   comments   a description of the property list.
     * @exception  IOException if writing this property list to the specified
     *             output stream throws an <tt>IOException</tt>.
     * @exception  ClassCastException  if this {@code Properties} object
     *             contains any keys or values that are not {@code Strings}.
     * @exception  NullPointerException  if {@code writer} is null.
     * @since 1.6
     */
    public void store(Writer writer, String comments)
        throws IOException
    {
        store0((writer instanceof BufferedWriter)?(BufferedWriter)writer
                                                 : new BufferedWriter(writer),
               comments,
               false);
    }

    /**
     * 将此{@code Properties}表中的此属性列表（键和元素对）以适合使用
     * {@link #load（InputStream）加载（InputStream）加载到
     * {@code Properties}表的格式写入输出流 } 方法。
     *  
     * 此{@code Properties}表的默认表中的属性（如果有）不是由此方法写出的。
     *  
     * 此方法以与{@link #store（java.io.Writer，java.lang.String）store（Writer）}
     * 中指定的格式相同的格式输出注释，属性键和值，但有以下区别：
     *  
     * 使用ISO 8859-1字符编码写入流。
     *
     * 注释中不在Latin-1中的字符被写为{@code \ u005Cu} xxxx，
     * 用于其适当的unicode十六进制值xxxx。
     *
     * 对于适当的十六进制值xxxx，属性键或值中小于{@code \ u005Cu0020}的字符和
     * 大于{@code \ u005Cu007E}的字符将写为{@code \ u005Cu} xxxx。
     *  
     *  
     * 写入条目后，将刷新输出流。 此方法返回后，输出流保持打开状态。
     * 
     * @param   out      an output stream.
     * @param   comments   a description of the property list.
     * @exception  IOException if writing this property list to the specified
     *             output stream throws an <tt>IOException</tt>.
     * @exception  ClassCastException  if this {@code Properties} object
     *             contains any keys or values that are not {@code Strings}.
     * @exception  NullPointerException  if {@code out} is null.
     * @since 1.2
     */
    public void store(OutputStream out, String comments)
        throws IOException
    {
        store0(new BufferedWriter(new OutputStreamWriter(out, "8859_1")),
               comments,
               true);
    }

    private void store0(BufferedWriter bw, String comments, boolean escUnicode)
        throws IOException
    {
        if (comments != null) {
            writeComments(bw, comments);
        }
        bw.write("#" + new Date().toString());
        bw.newLine();
        synchronized (this) {
            for (Enumeration<?> e = keys(); e.hasMoreElements();) {
                String key = (String)e.nextElement();
                String val = (String)get(key);
                key = saveConvert(key, true, escUnicode);
                /* No need to escape embedded and trailing spaces for value, hence
                 * pass false to flag.
                 */
                val = saveConvert(val, false, escUnicode);
                bw.write(key + "=" + val);
                bw.newLine();
            }
        }
        bw.flush();
    }

    /**
     * 将指定输入流上的XML文档表示的所有属性加载到此属性表中。
     *
     * XML文档必须具有以下DOCTYPE声明：
     *  
     * &lt;!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd"&gt;
     * </pre>
     * 此外，该文件必须满足上述特性DTD。
     *
     * 需要实现来读取使用“{@code UTF-8}”或“{@code UTF-16}”编码的XML文档。 
     * 实现可以支持其他编码。
     *
     * 此方法返回后，将关闭指定的流。
     *
     * @param in the input stream from which to read the XML document.
     * @throws IOException if reading from the specified input stream
     *         results in an <tt>IOException</tt>.
     * @throws java.io.UnsupportedEncodingException if the document's encoding
     *         declaration can be read and it specifies an encoding that is not
     *         supported
     * @throws InvalidPropertiesFormatException Data on input stream does not
     *         constitute a valid XML document with the mandated document type.
     * @throws NullPointerException if {@code in} is null.
     * @see    #storeToXML(OutputStream, String, String)
     * @see    <a href="http://www.w3.org/TR/REC-xml/#charencoding">Character
     *         Encoding in Entities</a>
     * @since 1.5
     */
    public synchronized void loadFromXML(InputStream in)
        throws IOException, InvalidPropertiesFormatException
    {
        XmlSupport.load(this, Objects.requireNonNull(in));
        in.close();
    }

    /**
     * 发出表示此表中包含的所有属性的XML文档。
     *
     * 调用props.storeToXML（os，comment）形式的此方法的行为与调用
     * props.storeToXML（os，comment，“UTF-8”）的行为完全相同;
     *
     * @param os the output stream on which to emit the XML document.
     * @param comment a description of the property list, or {@code null}
     *        if no comment is desired.
     * @throws IOException if writing to the specified output stream
     *         results in an <tt>IOException</tt>.
     * @throws NullPointerException if {@code os} is null.
     * @throws ClassCastException  if this {@code Properties} object
     *         contains any keys or values that are not
     *         {@code Strings}.
     * @see    #loadFromXML(InputStream)
     * @since 1.5
     */
    public void storeToXML(OutputStream os, String comment)
        throws IOException
    {
        storeToXML(os, comment, "UTF-8");
    }

    /**
     * 使用指定的编码发出表示此表中包含的所有属性的XML文档。
     *
     * XML文档将具有以下DOCTYPE声明：
     *  
     * &lt;!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd"&gt;
     * </pre>
     *
     * 如果指定的注释为{@code null}，则文档中不会存储任何注释。
     *
     * 需要实现来支持编写使用“{@code UTF-8}”或“{@code UTF-16}”编码的XML文档。
     * 实现可以支持其他编码。
     *
     * 此方法返回后，指定的流仍保持打开状态。
     *
     * @param os        the output stream on which to emit the XML document.
     * @param comment   a description of the property list, or {@code null}
     *                  if no comment is desired.
     * @param  encoding the name of a supported
     *                  <a href="../lang/package-summary.html#charenc">
     *                  character encoding</a>
     *
     * @throws IOException if writing to the specified output stream
     *         results in an <tt>IOException</tt>.
     * @throws java.io.UnsupportedEncodingException if the encoding is not
     *         supported by the implementation.
     * @throws NullPointerException if {@code os} is {@code null},
     *         or if {@code encoding} is {@code null}.
     * @throws ClassCastException  if this {@code Properties} object
     *         contains any keys or values that are not
     *         {@code Strings}.
     * @see    #loadFromXML(InputStream)
     * @see    <a href="http://www.w3.org/TR/REC-xml/#charencoding">Character
     *         Encoding in Entities</a>
     * @since 1.5
     */
    public void storeToXML(OutputStream os, String comment, String encoding)
        throws IOException
    {
        XmlSupport.save(this, Objects.requireNonNull(os), comment,
                        Objects.requireNonNull(encoding));
    }

    /**
     * 在此属性列表中搜索具有指定键的属性。 
     * 如果在此属性列表中找不到该键，则会检查默认属性列表及其默认值（递归）。 
     * 如果找不到该属性，则该方法返回{@code null}。
     *
     * @param   key   the property key.
     * @return  the value in this property list with the specified key value.
     * @see     #setProperty
     * @see     #defaults
     */
    public String getProperty(String key) {
        Object oval = super.get(key);
        String sval = (oval instanceof String) ? (String)oval : null;
        return ((sval == null) && (defaults != null)) ? defaults.getProperty(key) : sval;
    }

    /**
     * 在此属性列表中搜索具有指定键的属性。 
     * 如果在此属性列表中找不到该键，则会检查默认属性列表及其默认值（递归）。 
     * 如果未找到该属性，则该方法返回默认值参数。
     *
     * @param   key            the hashtable key.
     * @param   defaultValue   a default value.
     *
     * @return  the value in this property list with the specified key value.
     * @see     #setProperty
     * @see     #defaults
     */
    public String getProperty(String key, String defaultValue) {
        String val = getProperty(key);
        return (val == null) ? defaultValue : val;
    }

    /**
     * 返回此属性列表中所有键的枚举，如果尚未从主属性列表中找到相同名称的键，则
     * 包括默认属性列表中的不同键。
     *
     * @return  an enumeration of all the keys in this property list, including
     *          the keys in the default property list.
     * @throws  ClassCastException if any key in this property list
     *          is not a string.
     * @see     java.util.Enumeration
     * @see     java.util.Properties#defaults
     * @see     #stringPropertyNames
     */
    public Enumeration<?> propertyNames() {
        Hashtable<String,Object> h = new Hashtable<>();
        enumerate(h);
        return h.keys();
    }

    /**
     * 返回此属性列表中的一组键，其中键及其对应的值是字符串，如果尚未从主
     * 属性列表中找到相同名称的键，则包括默认属性列表中的不同键。 
     * 键或键不是String类型的属性将被省略。
     *  
     * 返回的集不受Properties对象的支持。 对此属性的更改不会反映在集合中，反之亦然。
     *
     * @return  a set of keys in this property list where
     *          the key and its corresponding value are strings,
     *          including the keys in the default property list.
     * @see     java.util.Properties#defaults
     * @since   1.6
     */
    public Set<String> stringPropertyNames() {
        Hashtable<String, String> h = new Hashtable<>();
        enumerateStringProperties(h);
        return h.keySet();
    }

    /**
     * 将此属性列表打印到指定的输出流。 此方法对于调试很有用。
     *
     * @param   out   an output stream.
     * @throws  ClassCastException if any key in this property list
     *          is not a string.
     */
    public void list(PrintStream out) {
        out.println("-- listing properties --");
        Hashtable<String,Object> h = new Hashtable<>();
        enumerate(h);
        for (Enumeration<String> e = h.keys() ; e.hasMoreElements() ;) {
            String key = e.nextElement();
            String val = (String)h.get(key);
            if (val.length() > 40) {
                val = val.substring(0, 37) + "...";
            }
            out.println(key + "=" + val);
        }
    }

    /**
     * 将此属性列表提取到指定的输出流。 此方法对于调试很有用。
     *
     * @param   out   an output stream.
     * @throws  ClassCastException if any key in this property list
     *          is not a string.
     * @since   JDK1.1
     */
    /*
     * 而不是使用匿名内部类来共享公共代码，此方法是重复的，以确保非1.1编译器
     * 可以编译此文件。
     */
    public void list(PrintWriter out) {
        out.println("-- listing properties --");
        Hashtable<String,Object> h = new Hashtable<>();
        enumerate(h);
        for (Enumeration<String> e = h.keys() ; e.hasMoreElements() ;) {
            String key = e.nextElement();
            String val = (String)h.get(key);
            if (val.length() > 40) {
                val = val.substring(0, 37) + "...";
            }
            out.println(key + "=" + val);
        }
    }

    /**
     * 枚举指定散列表中的所有键/值对。
     * @param h the hashtable
     * @throws ClassCastException if any of the property keys
     *         is not of String type.
     */
    private synchronized void enumerate(Hashtable<String,Object> h) {
        if (defaults != null) {
            defaults.enumerate(h);
        }
        for (Enumeration<?> e = keys() ; e.hasMoreElements() ;) {
            String key = (String)e.nextElement();
            h.put(key, get(key));
        }
    }

    /**
     * 枚举指定哈希表中的所有键/值对，如果键或值不是字符串，则省略该属性。
     * @param h the hashtable
     */
    private synchronized void enumerateStringProperties(Hashtable<String, String> h) {
        if (defaults != null) {
            defaults.enumerateStringProperties(h);
        }
        for (Enumeration<?> e = keys() ; e.hasMoreElements() ;) {
            Object k = e.nextElement();
            Object v = get(k);
            if (k instanceof String && v instanceof String) {
                h.put((String) k, (String) v);
            }
        }
    }

    /**
     * 将半字节转换为十六进制字符
     * @param   nibble  the nibble to convert.
     */
    private static char toHex(int nibble) {
        return hexDigit[(nibble & 0xF)];
    }

    /** A table of hex digits */
    private static final char[] hexDigit = {
        '0','1','2','3','4','5','6','7','8','9','A','B','C','D','E','F'
    };

    /**
     * 支持以XML格式加载/存储属性的类。
     *
     * 此处定义的{@code load}和{@code store}方法委托给系统范围的
     * {@code XmlPropertiesProvider}。 在首次调用任一方法时，系统范围的提供程序
     * 位于以下位置：
     *
     * <ol>
     *   如果定义了系统属性{@code sun.util.spi.XmlPropertiesProvider}，
     *   那么它将被视为具体提供程序类的完全限定名称。 该类加载了系统类加载器作为启
     *   动加载器。 如果无法使用零参数构造函数加载或实例化它，则会引发未指定的错误。
     *
     *   如果未定义系统属性，那么{@link ServiceLoader}类定义的服务提供者加载工具将
     * 用于定位一个提供程序，其中系统类加载器作为启动加载器和
     * {@code sun.util.spi.XmlPropertiesProvider} 作为服务类型。 
     * 如果此过程失败，则抛出未指定的错误。 
     * 如果安装了多个服务提供商，则不会指定将使用哪个提供商。
     *
     *   如果通过上述方式找不到提供者，则将实例化并使用系统默认提供者。
     * </ol>
     */
    private static class XmlSupport {

        private static XmlPropertiesProvider loadProviderFromProperty(ClassLoader cl) {
            String cn = System.getProperty("sun.util.spi.XmlPropertiesProvider");
            if (cn == null)
                return null;
            try {
                Class<?> c = Class.forName(cn, true, cl);
                return (XmlPropertiesProvider)c.newInstance();
            } catch (ClassNotFoundException |
                     IllegalAccessException |
                     InstantiationException x) {
                throw new ServiceConfigurationError(null, x);
            }
        }

        private static XmlPropertiesProvider loadProviderAsService(ClassLoader cl) {
            Iterator<XmlPropertiesProvider> iterator =
                 ServiceLoader.load(XmlPropertiesProvider.class, cl).iterator();
            return iterator.hasNext() ? iterator.next() : null;
        }

        private static XmlPropertiesProvider loadProvider() {
            return AccessController.doPrivileged(
                new PrivilegedAction<XmlPropertiesProvider>() {
                    public XmlPropertiesProvider run() {
                        ClassLoader cl = ClassLoader.getSystemClassLoader();
                        XmlPropertiesProvider provider = loadProviderFromProperty(cl);
                        if (provider != null)
                            return provider;
                        provider = loadProviderAsService(cl);
                        if (provider != null)
                            return provider;
                        return new jdk.internal.util.xml.BasicXmlPropertiesProvider();
                }});
        }

        private static final XmlPropertiesProvider PROVIDER = loadProvider();

        static void load(Properties props, InputStream in)
            throws IOException, InvalidPropertiesFormatException
        {
            PROVIDER.load(props, in);
        }

        static void save(Properties props, OutputStream os, String comment,
                         String encoding)
            throws IOException
        {
            PROVIDER.store(props, os, comment, encoding);
        }
    }
}


```