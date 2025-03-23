import opennlp.tools.doccat.DoccatModel;
import opennlp.tools.doccat.DocumentCategorizerME;
import opennlp.tools.tokenize.SimpleTokenizer;
import opennlp.tools.util.InputStreamFactory;
import opennlp.tools.util.MarkableFileInputStreamFactory;
import opennlp.tools.util.TrainingParameters;
import opennlp.tools.util.model.ModelUtil;
import java.io.File;
import java.io.FileOutputStream;
import java.io.OutputStream;
import java.nio.charset.StandardCharsets;
import java.util.HashMap;
import java.util.Map;
import java.util.Scanner;

public class Chatbot {
    private static DoccatModel model;

    public static void main(String[] args) {
        try {
            trainModel(); // Train the model
            Scanner scanner = new Scanner(System.in);
            System.out.println("Chatbot: Hello! How can I help you? (Type 'exit' to quit)");

            while (true) {
                System.out.print("You: ");
                String userInput = scanner.nextLine();
                if (userInput.equalsIgnoreCase("exit")) {
                    System.out.println("Chatbot: Goodbye!");
                    break;
                }
                String response = getResponse(userInput);
                System.out.println("Chatbot: " + response);
            }
            scanner.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    private static void trainModel() throws Exception {
        String[][] trainingData = {
            {"hello", "greeting"},
            {"hi", "greeting"},
            {"how are you", "greeting"},
            {"what is your name", "bot_name"},
            {"your name", "bot_name"},
            {"who are you", "bot_identity"},
            {"what do you do", "bot_identity"},
            {"bye", "farewell"},
            {"goodbye", "farewell"},
            {"thanks", "gratitude"},
            {"thank you", "gratitude"}
        };

        File trainingFile = new File("trainingData.txt");
        try (OutputStream os = new FileOutputStream(trainingFile)) {
            for (String[] data : trainingData) {
                os.write((data[0] + "\t" + data[1] + "\n").getBytes(StandardCharsets.UTF_8));
            }
        }

        InputStreamFactory inputStreamFactory = new MarkableFileInputStreamFactory(trainingFile);
        ObjectStream<String> lineStream = new PlainTextByLineStream(inputStreamFactory, StandardCharsets.UTF_8);
        ObjectStream<DocumentSample> sampleStream = new DocumentSampleStream(lineStream);

        model = DocumentCategorizerME.train("en", sampleStream, ModelUtil.createDefaultTrainingParameters(), new DoccatFactory());
        sampleStream.close();
    }

    private static String getResponse(String userInput) {
        DocumentCategorizerME categorizer = new DocumentCategorizerME(model);
        SimpleTokenizer tokenizer = SimpleTokenizer.INSTANCE;
        double[] outcomes = categorizer.categorize(tokenizer.tokenize(userInput));
        String category = categorizer.getBestCategory(outcomes);

        Map<String, String> responses = new HashMap<>();
        responses.put("greeting", "Hello! How can I assist you?");
        responses.put("bot_name", "I am a simple chatbot created using Java!");
        responses.put("bot_identity", "I am a natural language processing chatbot.");
        responses.put("farewell", "Goodbye! Have a great day.");
        responses.put("gratitude", "You're welcome!");

        return responses.getOrDefault(category, "I'm sorry, I didn't understand that.");
    }
}
