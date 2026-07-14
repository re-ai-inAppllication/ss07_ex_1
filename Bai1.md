# Bai 1: Phan tich va lua chon prompt viet ham Java xu ly XML

## 1. Dap an lua chon

Chon **phuong an B**.

## 2. Vi sao phuong an B toi uu nhat

Phuong an B la prompt tot nhat vi no mo ta ro vai tro, dau vao, dau ra, rang buoc xu ly, ngon ngu va yeu cau dry-run truoc khi sinh code.

| Thanh phan | Phan tich |
| :--- | :--- |
| Input | Neu ro dau vao la chuoi XML gom danh sach giao dich, moi giao dich co `id`, `amount`, `status`. |
| Processing | Yeu cau dung thu vien XML co san cua Java, validate `amount > 0`, `id` khong rong, va nem `InvalidTransactionException` khi du lieu sai. |
| Output | Yeu cau tra ve `List<TransactionDTO>` va dinh nghia DTO bang Java record. |
| Language | Chi ro Java 17, giup AI sinh code dung cu phap hien dai. |
| Dry-run CoT | Bat AI phac thao thuat toan, chay thu loi XML thieu the dong, sau do moi viet code. Dieu nay giam nguy co AI bo qua loi parsing va bien du lieu. |

## 3. Ly do loai tru cac phuong an con lai

### Phuong an A

Prompt qua ngan, thieu cau truc dau vao/dau ra, khong neu ro Java version, khong chi ro field cua `TransactionDTO`, khong yeu cau cach xu ly XML loi dinh dang. AI co the chi viet code chung chung, bat loi khong day du hoac bo qua case `amount <= 0`.

### Phuong an C

Sai trong tam bai toan. Prompt yeu cau parallel Stream, Spring XML Reader, bo qua dong loi va luu database, trong khi de bai can Java 17 core, validate chat che va nem exception. Viec bo qua dong loi lam mat du lieu va che giau sai sot nghiep vu.

## 4. Ma nguon Java hoan chinh

```java
import org.w3c.dom.Document;
import org.w3c.dom.Element;
import org.w3c.dom.Node;
import org.w3c.dom.NodeList;
import org.xml.sax.InputSource;
import org.xml.sax.SAXException;

import javax.xml.XMLConstants;
import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import javax.xml.parsers.ParserConfigurationException;
import java.io.IOException;
import java.io.StringReader;
import java.util.ArrayList;
import java.util.List;

public class TransactionXmlParser {

    public record TransactionDTO(String id, double amount, String status) {
    }

    public static class InvalidTransactionException extends Exception {
        public InvalidTransactionException(String message) {
            super(message);
        }

        public InvalidTransactionException(String message, Throwable cause) {
            super(message, cause);
        }
    }

    public List<TransactionDTO> parseTransactions(String xml)
            throws InvalidTransactionException {
        if (xml == null || xml.isBlank()) {
            throw new InvalidTransactionException("XML input must not be null or blank");
        }

        Document document = parseXml(xml);
        NodeList transactionNodes = document.getElementsByTagName("transaction");
        List<TransactionDTO> transactions = new ArrayList<>();

        for (int i = 0; i < transactionNodes.getLength(); i++) {
            Node node = transactionNodes.item(i);
            if (node.getNodeType() != Node.ELEMENT_NODE) {
                continue;
            }

            Element transactionElement = (Element) node;
            String id = getRequiredText(transactionElement, "id");
            String amountText = getRequiredText(transactionElement, "amount");
            String status = getRequiredText(transactionElement, "status");

            if (id.isBlank()) {
                throw new InvalidTransactionException("Transaction id must not be blank");
            }

            double amount = parseAmount(amountText, id);
            if (amount <= 0) {
                throw new InvalidTransactionException(
                        "Transaction amount must be greater than 0 for id: " + id);
            }

            transactions.add(new TransactionDTO(id, amount, status));
        }

        return transactions;
    }

    private Document parseXml(String xml) throws InvalidTransactionException {
        try {
            DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
            factory.setFeature(XMLConstants.FEATURE_SECURE_PROCESSING, true);
            factory.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
            factory.setExpandEntityReferences(false);
            factory.setNamespaceAware(false);

            DocumentBuilder builder = factory.newDocumentBuilder();
            return builder.parse(new InputSource(new StringReader(xml)));
        } catch (ParserConfigurationException | SAXException | IOException e) {
            throw new InvalidTransactionException("Invalid XML format", e);
        }
    }

    private String getRequiredText(Element parent, String tagName)
            throws InvalidTransactionException {
        NodeList nodes = parent.getElementsByTagName(tagName);
        if (nodes.getLength() == 0) {
            throw new InvalidTransactionException("Missing required tag: " + tagName);
        }
        return nodes.item(0).getTextContent().trim();
    }

    private double parseAmount(String amountText, String id)
            throws InvalidTransactionException {
        try {
            return Double.parseDouble(amountText);
        } catch (NumberFormatException e) {
            throw new InvalidTransactionException(
                    "Invalid amount for transaction id: " + id, e);
        }
    }
}
```

